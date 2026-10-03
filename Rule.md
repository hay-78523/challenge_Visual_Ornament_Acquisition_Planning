[LUẬT CỐ ĐỊNH]:
Tuyệt đối không Hardcode: Phát hiện và đánh dấu lỗi vi phạm nếu code có chứa các mảng/dict fix cứng kết quả đầu ra cho các tập test cụ thể nhằm lách luật. Cần giải pháp tổng quát.

Không dò dữ liệu ẩn (Dataset Fingerprinting): Bắt lỗi ngay lập tức nếu code cố tình in ra kích thước dữ liệu, đọc các biến môi trường, hoặc cố trích xuất ngược tập test ẩn.

Không lạm dụng API của LLM: Thí sinh phải tự viết thuật toán hoặc tự fine-tune mô hình. Bất kỳ hành vi import các thư viện như openai, anthropic, google.generativeai để gọi API giải quyết hộ bài toán đều bị nghiêm cấm.

Tối ưu hóa bằng thực lực: Phải sử dụng các kỹ thuật Machine Learning chuẩn (tối ưu loss function, epochs, learning rate...). Cấm dùng các thủ thuật if/else lách điểm số (MSE, RMSE).

Chất lượng code: Code là sản phẩm để mua lại đào tạo AI Agent. Đánh lỗi MINOR nếu code bẩn, thiếu logic, đặt tên biến vô nghĩa, không thể tái sử dụng.

Cấm Pseudo-labeling (gán nhãn giả): TUYỆT ĐỐI KHÔNG cho phép sử dụng pseudo-labeling (gán nhãn giả cho tập test rồi train lại, hoặc dùng prediction của model để gán nhãn rồi retrain). Vi phạm = CRITICAL.

Bắt buộc dùng Deep Learning / Pretrained Model: Nền tảng KHÔNG còn chấp nhận giải pháp chỉ dùng ML truyền thống (LightGBM, XGBoost, CatBoost, Random Forest, SVM, Logistic Regression...) làm mô hình chính. Thí sinh PHẢI sử dụng ít nhất 1 mô hình Deep Learning (CNN, Transformer, BERT, DeBERTa, Qwen, ViT, ResNet...) hoặc Pretrained Model làm backbone chính. ML truyền thống chỉ được phép dùng như mô hình phụ (ensemble, stacking, post-processing). Vi phạm = CRITICAL.

Cấm Regex / Biểu thức chính quy: TUYỆT ĐỐI KHÔNG cho phép code sử dụng regex hoặc biểu thức chính quy để so khớp/xử lý dữ liệu dưới bất kỳ hình thức nào, dù nhiều hay ít. Bắt lỗi nếu thấy import re, re.search, re.match, re.findall, re.sub, pattern regex trong string, hoặc logic so khớp dựa trên regular expression. Vi phạm = CRITICAL và phải REJECT/FAIL.

Cấm khai thác cấu trúc tập test (Test-Set Structural Leakage): Giải pháp phải xếp hạng/phân loại từng hàng test độc lập dựa trên model đã học từ train. TUYỆT ĐỐI CẤM dùng thông tin liên hàng trong tập test để ra quyết định, bao gồm gom nhóm/cluster/union-find các hàng test dựa trên co-occurrence trong metadata test; fit bất kỳ mô hình hoặc biến đổi thống kê nào trên dữ liệu test (PCA, normalization, KMeans, scaler, imputer, threshold/statistics learned from test...) rồi dùng kết quả đó làm basis cho prediction; hoặc để prediction của hàng A bị ảnh hưởng bởi dữ liệu của hàng B khi A và B là các test row độc lập. Quy tắc chung: mọi thứ được fit/learn phải đến từ train. Test chỉ được dùng để forward-pass qua model/pipeline đã cố định. Vi phạm = CRITICAL và phải REJECT/FAIL.

Cấm thư viện ngoài & truy cập Internet (Offline-only): Giải pháp CHỈ được dùng các thư viện đã cài sẵn trong môi trường Kaggle Docker. TUYỆT ĐỐI CẤM cài đặt package ngoài hoặc bất kỳ hành vi nào cần Internet lúc chạy, bao gồm: pip install / !pip install / gọi cài đặt qua subprocess, os.system, pip.main; tải file/trọng số/dataset qua mạng bằng requests, urllib, wget, curl, torch.hub.load(...), *.from_pretrained(...) tải online, huggingface_hub / hf_hub_download, gdown, kaggle API...; hoặc import các thư viện KHÔNG có sẵn trong Kaggle Docker. Trọng số pretrained CHỈ được load từ đường dẫn offline/local (Kaggle dataset đã mount sẵn), không được tải từ Internet. Vi phạm = CRITICAL và phải REJECT/FAIL.
<><>
For from-scratch challenges, avoid pretrained shortcuts unless the challenge clearly allows them. The point is to build features, train a model, validate it, and show that the model learned from the provided data. External embeddings, pretrained weights, internet downloads, or hidden data should be treated as not allowed by default.>For from-scratch challenges, avoid pretrained shortcuts unless the challenge clearly allows them. The point is to build features, train a model, validate it, and show that the model learned from the provided data. External embeddings, pretrained weights, internet downloads, or hidden data should be treated as not allowed by default.roject Eris: Solver Guidebook 🚀
Ranh giới dùng dữ liệu test

Mỗi packet/case test là một bài toán độc lập. Được phép khai thác mọi thông tin bên trong một packet (ví dụ 16 card của một bank): so sánh các card với nhau, xếp hạng, đếm, chạy EM hay tối ưu ràng buộc trong phạm vi đó, miễn là không có trạng thái nào mang từ packet này sang packet khác.

Không được phép để dữ liệu của packet này ảnh hưởng đến dự đoán của packet khác. Bao gồm: đếm tần suất hay đồng xuất hiện trên toàn tập test, lấy kết quả dự đoán của case khác làm feature (voting, stacking chéo), gộp xác suất role qua các case, decode kết hợp nhiều case, hoặc học lexicon/bảng tra từ cả list test. Loại trừ chính case đó ra (c2 != c) không làm việc này hợp lệ.

Học từ train thì luôn hợp lệ, kể cả khi áp bảng học được cho mọi case test.

Cách kiểm tra: chạy từng case test riêng lẻ, hoặc xáo thứ tự tập test. Nếu dự đoán của bất kỳ case nào thay đổi thì bài vi phạm.

1.Welcome to Project Eris
1.1 What Eris Is Trying to Measure
    <> While creating solutions, please keep in mind that your solutions - at some stage - will be used to train SOTA models / Agents. Hence, ensure that your solutions involve real ML training / finetuning. Inference only solutions are not allowed. If you have found anything important like a genuine insight in the dataset, instead of hardcoding it, consider training a model to find and use that insight. Because when your solution goes to the Agent, that agent will learn how to train a model to replicate that behaviour. But if it sees anything hard coded, it doesn’t understand what led to it, how to do it on another problem, and so on. 

    For example, consider 2 cases:
        1. You are using HPO (Hyper parameter Optimization) techniques within the submission script to optimize and find the best set of parameters for your model / solution. 
    2. You are hardcoding the best set of parameters you manually searched offline, locally. 
Looking at a solution from case 1, the Agent understands and genuinely learns the concept, importance, and best practices of HPO. However, looking at a solution from case 2, the agent cannot infer any knowledge. It cannot find any pattern saying that “on such a challenge I have to use these sets of parameters“. It's really as dumb as it sounds. 
1.2 Solving the Challenge, Not Chasing the Leaderboard 
    <> Note that the platform acts just like Kaggle. It has a Public LB and a Private LB. As most of you might already know, Private LB is the Final Leaderboard considered for prizes. Not the Public LB. However, it's hidden throughout the competition and is only revealed at the end of the challenge. So during the competition itself, you will only be able to see the Public LB. But don’t take that score or rank as a True representative of your solution’s generalizability. If you over optimize on Public LB blindly, your Final rank might drop drastically. So use Public LB ranks just for a soft indication of how good your solution is doing on test data. Not as its true generalizability.

quick note: Eris is not trying to collect leaderboard hacks. The point is to make future ML agents better at solving ML challenges with actual ML. So even if a task can be cracked with a clever rule, lookup, regex, or hand-built algorithm, that is usually not what we want. A good solution should show a learnable process: inspect the data, build validation, train or fine-tune something, improve it, and explain why it should generalize. 


2.Guide to Solve an Eris Challenge
Start simple. Read the full challenge first, especially the metric, submission format, data rules, and what not to use. Then inspect the files before modeling: columns, ids, labels, missing values, duplicates, and the sample submission.

Build a boring valid baseline first. After that, make a local validation split that feels close to the hidden/private test set. Use the public leaderboard as a sanity check, not as your only validation.

A strong solution usually looks like this: understand the task, make a baseline, train or fine-tune a real model, compare it on validation, improve it, check the submission format, then explain the approach honestly.
3.General Solver Rules
3.1 Clearly Allowed
Generally safe stuff:

1. training models on the public train data
2. making a local validation split
3. normal data cleaning, parsing, tokenization, resizing, and augmentation
4. using standard ML models like linear models, trees, boosting, neural nets, embeddings, or transformers when the challenge allows them
5. hyperparameter tuning when it is done inside the solution process, not secretly searched offline and hardcoded
6. ensembling valid models
7. simple post-processing to make outputs valid, like clipping ranges or fixing JSON format
8. using external data or pretrained models only if the challenge clearly allows it
9. Your solution must finish running in 1 hour, plus 5-10 minutes for buffer, meaning total runtime is 60-70 minutes.
3.2 Clearly Not Allowed
Not okay:

1. using private answers, reviewer files, hidden labels, creator files, or platform internals
2. hardcoding predictions for test ids, row order, filenames, or public LB behavior
3. exploiting grader bugs, leaked targets, metadata leaks, or source reconstruction
4. using a lookup table or rule system that skips the actual learning problem
5. using external data, APIs, internet downloads, or unsupported packages when the challenge does not allow them
6. submitting something that is mostly handcrafted shortcuts while describing it like a model-based solve
7. repeatedly probing the public leaderboard just to fit it
8. Using pseudo-labelling, or using test data to train it prohibited
3.3 Grey-Area-Techniques

These are some of the Grey-area methods that should only be used when the solver accepts the risk that the solution may be rejected.
Quick rule for grey areas: regex, TF-IDF, BM25, keyword rules, and other heuristics are not automatically bad. They are fine when they help clean data, parse structured fields, guard LLM outputs, or support a real model. They become a problem when they are the main shortcut and bypass the intended ML solve.


1.Using regex in your solutions. (Especially in NLP competitions): Using regex itself is not a violation. But you have to keep in mind that over reliance on Regex and similar techniques make the solution too fragile. The solution is NOT robust/generalizable anymore. Hence, use regexes as little as possible. Using it for cleaning the dataset, finding something perfectly deterministic like numbers, etc are allowed. But if we catch that your solution is overrelying on regex and not ML, like exploiting how the data was generated, overfitting to the underlying distribution of the data, and so on, we will reject it.  
2.Using TF-IDF in your solutions: Just like Regex, this again falls into Grey Area. Its fully up to the Reviewer of that competition whether he wants to allow the usage of it or not. Also note that, the Reviewer has full rights to allow the usage of TF-IDF (and other such vectorizers) for one participant, and not for the other within the same competition. It mainly depends on 2 factors. The quality/novelty of your solutions, and how much is it relying on TF-IDF. So only use it if you agree with the rules. 
3.4 Challenge-Specific Rules
Challenge-specific rules always win. If a challenge bans a method, data source, package, pretrained model, internet access, or inference-only solve, follow that even if this guidebook sounds more flexible. When in doubt, ask before submitting instead of taking the risky interpretation.
4.Domain-Specific Guidelines
4.1 NLP Challenges
For NLP, the solution should learn from the text, not just memorize keywords. Cleaning, tokenization, TF-IDF baselines, embeddings, fine-tuning, and validation splits are all fine when the challenge allows them. Be careful with regex-only classifiers, template matching, duplicate leakage, or anything that mainly exploits how the text was generated. Also, using NN/pretrained as the main predictor, do not use entire trees, gradient boosting, tabular models, etc as the main, it should serve as a layer of the NN/pretrained model to process some features.
4.2 Computer Vision and Object Detection Challenges
For CV or object detection, make sure the model is learning from pixels and annotations, not filenames, metadata, or fixed coordinates. Normal resizing, augmentation, fine-tuning, NMS, box-format checks, and confidence thresholds are fine. Watch for duplicate images, leaked labels, broken box formats, or external weights that the challenge did not allow.
4.3 RAG and Retrieval Challenges
For RAG and retrieval, the solve should retrieve useful evidence and use it correctly. BM25, embeddings, rerankers, hybrid search, and chunking are all okay when built from allowed documents. Be careful with hidden corpora, answer lookup, source reconstruction, or retrieval that only works because the test examples expose the source pattern.
4.4 Fine-tuning Challenges
For fine-tuning, the solution should actually train or adapt a model using the provided data. LoRA, small model fine-tuning, validation-based early stopping, prompt formatting, and output-format cleanup are all fine when allowed. A pure prompt-only or inference-only solution is risky if the challenge expects training.
4.5 From-Scratch Challenges
<>For from-scratch challenges, avoid pretrained shortcuts unless the challenge clearly allows them. The point is to build features, train a model, validate it, and show that the model learned from the provided data. External embeddings, pretrained weights, internet downloads, or hidden data should be treated as not allowed by default.


Các thư viện có thể dùng được
jax
jax-cuda12-pjrt
jax-cuda12-plugin
jaxlib
keras
keras-hub
keras-nlp
tensorflow
tensorflow-datasets
tensorflow-hub
tensorflow-metadata
tensorflow-probability
tensorflow-text
tensorflow_decision_forests
tf_keras
torch
torchao
torchaudio
torchdata
torchsummary
torchtune
torchvision
Boruta
Cartopy
ImageHash
Janome
PyArabic
PyUpSet
Pympler
Rtree
shapely
SimpleITK
TPOT
Wand
bayesian-optimization
boto3
catboost
category-encoders
cesium
comm
cytoolz
datasets
deap
dipy
docker
easyocr
emoji
fasttext
featuretools
fiona
fury
fuzzywuzzy
geojson
gensim
google-adk[a2a,eval]
google-cloud-aiplatform
google-cloud-videointelligence
google-cloud-vision
google-genai
gpxpy
h2o
haversine
hep-ml
igraph
ipympl
ipywidgets
isoweek
jedi
jiter
jupyter-lsp
jupyter_server
jupyter_server_proxy
jupyterlab
jupyterlab-lsp
kaggle
kaggle-environments
keras-cv
keras-nlp
keras-tuner
kornia
langid
libpysal
lime
line_profiler
mamba
matplotlib
mlcrate
mne
mpld3
nbconvert
nbdev
nilearn
olefile
onnx
openslide-bin
openslide-python
optuna
pandas-profiling
pandasql
papermill
path
path.py
pdf2image
plotly-express
preprocessing
pudb
pyLDAvis
pycryptodome
pydicom
pyemd
pyexcel-ods
pymongo
pypdf
pytesseract
python-lsp-server
pytorch-ignite
pytorch-lightning
qgrid
qtconsole
ray
rgf-python
s3fs
scikit-learn
scikit-learn-intelex
scikit-multilearn
scikit-optimize
scikit-plot
scikit-surprise
git+https://github.com/facebookresearch/segment-anything.git
squarify
tensorflow-cloud
tensorflow-io
torchinfo
torchmetrics
torchtune
transformers
vtk
wavio
xvfbwrapper
ydata-profiling


Một số lưu ý từng bị reject

N-gram Decoding
Randomly initialized network trained from scratch, violates the Fine-Tuning requirement
