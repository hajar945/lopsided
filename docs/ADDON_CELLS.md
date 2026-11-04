# Optional add-on cells for `lopsided_v4h.ipynb`

**How to use:** open `lopsided_v4h.ipynb` on a **CPU runtime** with `QUICK = False`, click **Run all**, and wait for it to finish. Then add a new cell at the very bottom and paste ONE block below. These cells need the variables the notebook has just created, so they must run in the **same session** as Run all (if Colab disconnects, run all again first; finished work reloads in minutes). Nothing here edits or overwrites the existing results or checkpoints.

| Cell | Addresses | Time (CPU) |
|---|---|---|
| A | Package versions not pinned | 2 seconds |
| B | "Equivalent" verdict depends on the +/-0.05 margin | 5 seconds |
| C | Brain mask is about twice a real brain | 20 seconds |
| D | "Satellite lesion" analysis that was dropped | 30 seconds |
| E | Trust flag: is the size rule really better than the disagreement flag? | 10 seconds |
| F | Small-training-size results use one random draw | 1 to 1.5 hours |
| G | Ablation has no confidence interval | about 40 minutes |
| H | Original statement promised sensitivity; we only reported Dice | 5 seconds |
| I | Performance by sex, age and stroke severity (subgroup check) | 5 seconds |
| J | Headline uses mean patient Dice; does a volume-weighted (pooled) Dice change the order? | about 1 minute |

Do A to E, H and I first (about 2 minutes in total). F and G are optional strengtheners.

---

## Cell A: pin the package versions (copy the printout into `requirements.txt`)
```python
import importlib.metadata as imd, sys
print('python', sys.version.split()[0])
for p in ['nibabel', 'pyarrow', 'pandas', 'numpy', 'scipy', 'scikit-learn', 'xgboost', 'shap', 'torch', 'matplotlib', 'tqdm', 'psutil']:
    try: print(f'{p}=={imd.version(p)}')
    except Exception: print(f'# {p}: not installed in this session')
```

## Cell B: does the "equivalent" verdict survive a stricter margin?
```python
print('=== xgb minus U-Net, paired bootstrap over patients, verdict at three margins ===')
for deep in [d for d in ('unet_raw', 'unet_mirror') if d in set(main.model)]:
    m, lo, hi = paired_ci(pat_dice(main, 'xgb'), pat_dice(main, deep))
    print(f'full data, xgb - {deep}: {m:+.3f} [{lo:+.3f}, {hi:+.3f}]')
    for delta in (0.02, 0.03, 0.05): print(f'    margin +/-{delta:.2f}: {verdict(lo, hi, delta)}')
if best_unet:
    print(f'\nBy training size, xgb - {best_unet}:')
    for n in sorted(curve['n_lab'].unique()):
        a = curve[(curve.model == 'xgb') & (curve.n_lab == n)].set_index('subject_id')['dice']
        b = curve[(curve.model == best_unet) & (curve.n_lab == n)].set_index('subject_id')['dice']; k = a.index.intersection(b.index)
        m, lo, hi = paired_ci(a[k], b[k])
        print(f'  n={n:4d} {m:+.3f} [{lo:+.3f}, {hi:+.3f}] | ' + ' | '.join(f'+/-{d:.2f}: {verdict(lo, hi, d).split(" (")[0]}' for d in (0.02, 0.03, 0.05)))
```

## Cell C: show the brain mask on three patients (turns a hidden flaw into an explained limitation)
```python
fig, axes = plt.subplots(3, 3, figsize=(11, 11))
for r, sid in enumerate(IDS[:3]):
    g = get(sid); zs = np.nonzero(g['brain'].any(axis=(0, 1)))[0]; span = zs.max() - zs.min()
    for c, z in enumerate((zs.min() + span // 5, (zs.min() + zs.max()) // 2, zs.max() - span // 5)):
        axes[r, c].imshow(g['arr'][0][:, :, z].T.astype(np.float32), cmap='jet', origin='lower', vmin=0, vmax=60)
        axes[r, c].contour(g['brain'][:, :, z].T.astype(float), levels=[0.5], colors='white', linewidths=1)
        axes[r, c].set_title(f'{sid}  slice {z}'); axes[r, c].axis('off')
plt.suptitle('White outline = the brain mask used by every model (blood-flow map underneath)'); plt.tight_layout(); plt.show()
print('Mean mask volume: %.0f mL | a real adult brain is roughly 1,300 mL' % np.mean([FEAT[s]['n'] * FEAT[s]['vox_ml'] for s in IDS]))
```

## Cell D: the dropped "satellite lesion" analysis (are lesions in separate pieces harder?)
```python
import scipy.ndimage as ndi
rows = []
for s in IDS:
    m = get(s)['mask']; lab, n = ndi.label(m)
    if n == 0: continue
    sizes = np.bincount(lab.ravel())[1:]; vm = FEAT[s]['vox_ml']
    rows.append(dict(subject_id=s, n_pieces=n, total_ml=sizes.sum() * vm, satellite_frac=1 - sizes.max() / sizes.sum()))
S = pd.DataFrame(rows).set_index('subject_id'); S['dice'] = main[main.model == 'xgb'].groupby('subject_id')['dice'].mean().reindex(S.index)
S['has_satellites'] = S['satellite_frac'] >= 0.10
print(f'{int(S.has_satellites.sum())} of {len(S)} patients have >=10% of their lesion in pieces separate from the main lesion (median pieces per patient: {int(S.n_pieces.median())})')
print('\nxgb Dice by satellite status:'); print(S.groupby('has_satellites')['dice'].agg(['mean', 'count']).round(3).to_string())
big = S[S.total_ml >= S.total_ml.median()]
print('\nWithin larger lesions only (to separate the satellite effect from lesion size):'); print(big.groupby('has_satellites')['dice'].agg(['mean', 'count']).round(3).to_string())
```

## Cell E: is the lesion-size rule really a better trust flag than model disagreement?
```python
if best_unet is None: print('Needs the U-Net results (they reload from the checkpoints on Run all).')
else:
    d, s_dis, s_size = T['dice'].values, T['score'].values, -T['vol_x'].values
    def gain(sc, i, c=0.6):
        k = max(3, int(round(c * len(i)))); o = np.argsort(sc[i], kind='stable')[:k]; return d[i][o].mean() - d[i].mean()
    rng = np.random.default_rng(0); n = len(T); allidx = np.arange(n)
    diffs = np.array([gain(s_size, i) - gain(s_dis, i) for i in (rng.integers(0, n, n) for _ in range(2000))]); lo, hi = np.percentile(diffs, [2.5, 97.5])
    print(f'Gain at 60% coverage: size rule {gain(s_size, allidx):+.3f} vs disagreement flag {gain(s_dis, allidx):+.3f}')
    print(f'Difference (size minus disagreement): {diffs.mean():+.3f}, 95% CI {lo:+.3f} to {hi:+.3f}')
    print('Interval excludes 0 -> the size rule is genuinely better. Includes 0 -> call the two flags comparable.')
```

## Cell F (optional, about 1 to 1.5 hours): extra random draws of the training patients for the simple model and the threshold
Stress-tests the n = 10 to 40 findings (the least certain part of the headline). The U-Net stays at one draw, because it needs a GPU.
```python
SEEDS_EXTRA = (1, 2)       # extra random draws (the main run used one draw per fold)
N_EXTRA = [10, 20, 40]     # n = 80 is slow and adds little
CKX = os.path.join(PROJECT, f'ck_extra_seeds_{NB_VER}_ds{DS}.pkl')
ckx = pickle.load(open(CKX, 'rb')) if os.path.exists(CKX) else dict(rows=[], done=set())
for f in FOLDS_TO_RUN:
    tr_all = [s for s in IDS if FOLD[s] != f]; te = [s for s in IDS if FOLD[s] == f]
    for n in N_EXTRA:
        for seed in SEEDS_EXTRA:
            rng = np.random.default_rng(100 * seed + f); tr = list(rng.choice(tr_all, n, replace=False))
            for name in ('rule_best', 'xgb'):
                if (f, n, seed, name) in ckx['done']: continue
                rows, _ = run_model(name, tr, te, seed, hp=BEST_HP.get(name))
                for r in rows: r.update(model=name, fold=f, n_lab=n, seed=seed)
                ckx['rows'] += rows; ckx['done'].add((f, n, seed, name)); pickle.dump(ckx, open(CKX, 'wb'))
    print('fold', f, 'done')
X = pd.DataFrame(ckx['rows']); orig = de[de.model.isin(['rule_best', 'xgb']) & de.n_lab.isin(N_EXTRA)][['model', 'n_lab', 'subject_id', 'dice', 'seed']]
pp = pd.concat([orig, X[['model', 'n_lab', 'subject_id', 'dice', 'seed']]]).groupby(['model', 'n_lab', 'subject_id'])['dice'].mean().reset_index()
print(f'\nxgb minus best single threshold, averaged over {1 + len(SEEDS_EXTRA)} random draws of the training patients:')
for n in N_EXTRA:
    a = pp[(pp.model == 'xgb') & (pp.n_lab == n)].set_index('subject_id')['dice']; b = pp[(pp.model == 'rule_best') & (pp.n_lab == n)].set_index('subject_id')['dice']; k = a.index.intersection(b.index)
    m, lo, hi = paired_ci(a[k], b[k]); print(f'  n={n:3d}: {m:+.3f} [{lo:+.3f}, {hi:+.3f}] -> ' + ('xgb ahead' if lo > 0 else 'xgb behind' if hi < 0 else 'tied'))
```

## Cell G (optional, about 40 minutes): ablation with confidence intervals
Re-runs the "all features" and "without any left-right comparison" models keeping per-patient scores, so the 0.010 difference gets an interval. It does not touch the earlier ablation results.
```python
AB2 = os.path.join(PROJECT, f'ck_abl_perpatient_{NB_VER}_ds{DS}.pkl')
ab2 = pickle.load(open(AB2, 'rb')) if os.path.exists(AB2) else {}
CFG = [('all features', lambda n: True), ('without ANY left-right comparison', lambda n: 'mir' not in n)]   # add ('without position', lambda n: FEAT_GROUP[n] != 'position') for a third (+20 min)
for label, keep_fn in CFG:
    if label in ab2: continue
    cols = [i for i, n in enumerate(FEAT_NAMES) if keep_fn(n)]; res = {}
    for f in FOLDS_TO_RUN:
        tr = [s for s in IDS if FOLD[s] != f]; te = [s for s in IDS if FOLD[s] == f]; M = fit_gbm(tr, SEED, BEST_HP['xgb'], cols)
        for s in te: res[s] = score_pred(predict_p(M['model'], s, M['cols']) > M['thr'], s)[0]
    ab2[label] = pd.Series(res); pickle.dump(ab2, open(AB2, 'wb')); print('done:', label, round(ab2[label].mean(), 3))
base = ab2['all features']
for label in [l for l, _ in CFG[1:]]:
    m, lo, hi = paired_ci(base, ab2[label].reindex(base.index))
    print(f'{label}: change in mean Dice {-m:+.3f} (95% CI {-hi:+.3f} to {-lo:+.3f})   [negative = the model got worse]')
```

## Cell H: sensitivity and precision for every model (the original statement asked for sensitivity)
Derived exactly from the stored Dice and volumes (Dice = 2 x overlap / (predicted + true), so the overlap, and from it sensitivity and precision, can be recovered). No retraining. Voxel-level ROC-AUC needs saved probabilities and is not available.
```python
m = main.copy()
m['tp'] = m['dice'] * (m['vol_pred'] + m['vol_true']) / 2
m['sens'] = (m['tp'] / m['vol_true']).where(m['vol_true'] > 0); m['prec'] = (m['tp'] / m['vol_pred']).where(m['vol_pred'] > 0)
g = m.groupby('model')
out = pd.DataFrame({'mean Dice': g['dice'].mean(), 'sensitivity (mean of patients)': g['sens'].mean(), 'precision (mean of patients)': g['prec'].mean(),
                    'sensitivity (all voxels pooled)': g['tp'].sum() / g['vol_true'].sum(), 'precision (all voxels pooled)': g['tp'].sum() / g['vol_pred'].sum()}).sort_values('mean Dice', ascending=False)
print(out.round(3).to_string())
print('\nSensitivity = share of the expert-marked lesion the model found. Precision = share of what the model marked that really was lesion.')
```

## Cell I: performance by sex, age and admission stroke severity
Groups can differ just because their lesions differ in size, so the median lesion size is printed too. Small groups: read as a check, not a finding.
```python
Q = pd.DataFrame([dict(subject_id=s, age=META[s].get('age'), sex=META[s].get('sex'), nihss=META[s].get('nihss_admission')) for s in IDS]).set_index('subject_id')
Q['age'] = pd.to_numeric(Q['age'], errors='coerce'); Q['nihss'] = pd.to_numeric(Q['nihss'], errors='coerce')
X_ = main[main.model == 'xgb'].groupby('subject_id').agg(dice=('dice', 'mean'), lesion_ml=('vol_true', 'first'))
Q = Q.join(X_)
Q['age_group'] = pd.qcut(Q['age'].rank(method='first'), 3, labels=['youngest third', 'middle third', 'oldest third'])
Q['severity'] = pd.qcut(Q['nihss'].rank(method='first'), 3, labels=['mildest third', 'middle third', 'most severe third'])
for col in ('sex', 'age_group', 'severity'):
    t = Q.groupby(col, observed=True).agg(mean_dice=('dice', 'mean'), median_lesion_mL=('lesion_ml', 'median'), patients=('dice', 'count')).round(3)
    print(f'--- interpretable model, Dice by {col} ---'); print(t.to_string()); print()
print('Age ranges:', Q.groupby('age_group', observed=True)['age'].agg(['min', 'max']).to_dict('index'))
```

---

## Cell J: does the order change when large lesions count more? (pooled Dice with intervals)
Cell H's pooled sensitivity and precision imply a higher pooled Dice for the U-Nets than for the interpretable model, the opposite of mean patient Dice. This tests it. Stored results only; no retraining. Run after Run all, same session.
```python
m = main.copy()
m['tp'] = m['dice'] * (m['vol_pred'] + m['vol_true']) / 2
P = m.groupby(['model', 'subject_id'])[['tp', 'vol_pred', 'vol_true']].sum()
def pooled(model, idx):
    d = P.loc[model].reindex(idx)
    return 2 * d['tp'].sum() / (d['vol_pred'].sum() + d['vol_true'].sum())
ids = np.array(sorted(set(m.subject_id))); rng = np.random.default_rng(0)
boots = [rng.choice(ids, len(ids)) for _ in range(2000)]
print('Pooled Dice (all voxels together; big lesions dominate):')
for mod in P.index.get_level_values(0).unique(): print(f'  {mod:12s} {pooled(mod, ids):.3f}')
for deep in [d for d in ('unet_mirror', 'unet_raw', 'rule_best') if d in P.index.get_level_values(0)]:
    obs = pooled('xgb', ids) - pooled(deep, ids)
    diffs = np.array([pooled('xgb', i) - pooled(deep, i) for i in boots]); lo, hi = np.percentile(diffs, [2.5, 97.5])
    print(f'xgb - {deep}: {obs:+.3f} [{lo:+.3f}, {hi:+.3f}] ->', 'xgb ahead' if lo > 0 else 'xgb behind' if hi < 0 else 'tied (CI includes 0)')
print('Compare with mean patient Dice: xgb - unet_mirror +0.011, xgb - unet_raw +0.015.')
```

**Cell J result (full data):** pooled Dice xgb 0.322, mirror U-Net 0.340, raw U-Net 0.365, best threshold 0.257. xgb minus raw U-Net -0.043 [-0.074, -0.010]; minus mirror U-Net -0.018 [-0.047, +0.014]; minus best threshold +0.065 [+0.040, +0.088].
