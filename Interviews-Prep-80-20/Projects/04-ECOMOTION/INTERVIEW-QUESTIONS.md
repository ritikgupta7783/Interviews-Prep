# EcoMotion — 80/20 Interview Questions

**Resume framing:** Group project; environmental intelligence for climate/vegetation monitoring; Python, machine learning, REST APIs and Vercel; first prize at the NIT Silchar Eco Club Hackathon.

**Source inspected:** public repository `Tahmid019/Hyperspectral_dataAnalysis`, especially `tests/HyperSpectral5g.ipynb`, crop-detection metrics CSVs/plot and saved model weights. Live demo: https://ecomotionhack.vercel.app/.

**Critical evidence boundary:** The notebook demonstrates hyperspectral ML experiments. It does not, by itself, prove that every notebook model is connected to the deployed web demo or that the broader IoT/LoRaWAN/MQTT architecture is implemented. Do not merge intended architecture with verified code.

## P0 — Must answer confidently

### 1. Explain EcoMotion in 60 seconds.
**Answer cue:** “EcoMotion is a climate and vegetation intelligence project built for environmental monitoring. My resume describes a full-stack platform with backend APIs for collecting/processing data and a dashboard. The public analysis repository includes hyperspectral reflectance processing and crop-classification experiments using Python ML tools. Our project won first prize at the NIT Silchar Eco Club Hackathon.” Then state your personal contribution accurately.

### 2. What is hyperspectral imaging?
**Answer cue:** Each pixel contains measurements across many narrow wavelength bands, producing a 3D cube: height × width × spectral bands. Unlike an RGB image, it contains richer spectral signatures that can help distinguish vegetation/material properties. Explain that the notebook's Indian Pines cube is recorded as 145 × 145 × 200.

### 3. What does a hyperspectral data cube look like in the code?
**Answer cue:** `hsi` has shape `(145, 145, 200)`. The first two axes are spatial coordinates; the last axis is spectral bands. `hsi.reshape(-1, C)` turns pixels into rows and bands into features, giving `(21025, 200)`. Ground-truth labels are flattened to match pixels.

### 4. Explain the preprocessing pipeline.
**Answer cue:** The notebook uses Savitzky–Golay smoothing, log transformation, missing-value handling, PCA and vegetation-index calculation. Smoothing reduces spectral noise; log transforms can compress dynamic range; imputation handles missing values; PCA compresses correlated bands; NDVI-like index summarizes red/NIR contrast. Important: verify training/inference use exactly the same fitted transformations.

### 5. What is PCA and why use it?
**Answer cue:** Principal Component Analysis projects correlated features into fewer orthogonal components that retain as much variance as possible. It can reduce dimensionality and model cost, but it may discard task-relevant information and components are less interpretable. Fit PCA on training data only, then transform validation/test data.

### 6. What is NDVI?
**Answer cue:** A vegetation index based on near-infrared and red reflectance: `(NIR - Red) / (NIR + Red)`, with safeguards for a zero denominator. The notebook calculates an index from selected spectral positions; the exact band-to-wavelength mapping must be verified for the dataset before claiming precise wavelengths.

### 7. Which models are actually present in the notebook?
**Answer cue:** It contains Random Forest experiments, a PyTorch feed-forward neural network for classifying ground-truth labels, several enhanced crop classifiers, and a proposed multitask architecture with nutrient regression, disease classification and crop embeddings. A class definition is not proof that a model was trained/evaluated or deployed.

### 8. What does the recorded ~66.7% validation accuracy mean?
**Answer cue:** One recorded `NutrientNN_1` classification experiment on the eggplant reflectance/ground-truth data reached a best validation accuracy around 0.6669 over ten epochs. This is not overall EcoMotion accuracy. To interpret it properly, state dataset, labels, preprocessing, split method, class balance, baseline, metric and whether validation leakage exists.

### 9. What is a major ML evaluation concern in the notebook?
**Answer cue:** In `train_model_DL1`, scaling is fit on the full dataset before the train/validation split, which can leak validation distribution information. Split first, fit scaler/PCA on training only, transform validation/test with those fitted objects, and evaluate once on held-out test data. Also consider spatial leakage because nearby pixels can be highly correlated.

### 10. How do you connect the ML notebook to the deployed app?
**Answer cue:** Be honest. The public source tree confirms notebook experiments and saved model artifacts, but that alone does not verify an end-to-end link to the live demo. Identify the actual backend endpoint, model-loading/inference code, request/response contract and deployment artifact if present. If not traceable, say integration needs verification.

## P1 — Strong follow-up questions

11. Why is a spectral signature useful for crop/vegetation analysis?
12. What are labels/ground truth, and how can label quality affect model performance?
13. What is the difference between classification and regression in the proposed multitask model?
14. Why use a Random Forest baseline before a deep model?
15. What are overfitting and underfitting? How would you detect them from loss/accuracy curves?
16. Why are class imbalance and macro-F1 relevant for crop/disease classification?
17. What does a confusion matrix reveal that accuracy hides?
18. Why can random pixel splitting overestimate generalization for spatial imagery?
19. How would you choose a spatially separated train/test split?
20. What is the role of a CNN over spectral sequences, and what does a Transformer add?
21. What does `nn.Conv1d` expect for tensor dimensions?
22. Why is a shared backbone plus multiple task heads a multitask design?
23. What is Huber loss, and why might it be used for nutrient regression?
24. What is triplet loss, and what are anchor/positive/negative examples?
25. Why can `torch.quantization.quantize_dynamic` help deployment, and what accuracy/performance trade-offs must be measured?
26. What is ONNX and when would exporting to it help?
27. Why does hard-coded `.cuda()` reduce portability?
28. How would you handle NaNs, negative reflectance or zero denominators safely?
29. How would you version model weights and preprocessing objects together?
30. How would you expose model inference through a REST API and validate the input dimensions?
31. What is the difference between inference latency and model accuracy?
32. What would you monitor after deployment: input drift, prediction distribution, errors, latency or accuracy?
33. What is the role of Vercel in the project? Which parts are frontend versus backend/model hosting?
34. What exactly did you personally implement in this group project?
35. Which design decision helped your team win the hackathon, and what evidence supports that explanation?

## P2 — Advanced but relevant

36. What is data leakage, and how can preprocessing cause it?
37. How would you design an experiment comparing Random Forest, MLP and CNN/Transformer fairly?
38. Why are per-pixel splits risky for spatially correlated datasets?
39. How would you handle different sensors with different band counts/wavelength calibration?
40. What if a deployment receives a spectrum with 277 bands but a model expects 200?
41. How would you optimize inference for edge devices with limited CPU/RAM?
42. How would you quantify uncertainty rather than returning only a hard class?
43. How would you test model robustness across seasons, lighting and geography?
44. What evidence would you need before claiming the model can predict actual nutrient concentration?
45. If the model says “disease detected,” how would you communicate uncertainty and avoid unsupported agronomic advice?

## Technical honesty checklist

- Do not call a proposed multitask model “trained” unless you can show training/evaluation evidence for each head.
- Do not use 66.7% as the platform's overall accuracy.
- Do not claim IoT, LoRaWAN, MQTT, edge deployment or cloud ingestion is implemented without tracing the code/hardware/demo.
- Be ready to explain your own contribution separately from teammates' work.
