# IEEE-CIS Fraud Detection

## Kaggle-ის კონკურსის მოკლე მიმოხილვა

ეს პროექტი მუშაობს Kaggle-ის ამოცანაზე [IEEE-CIS Fraud Detection](https://www.kaggle.com/competitions/ieee-fraud-detection). მონაცემები შეიცავს ბევრი ტრანზაქციის ინფორმაციას, სადაც 433 feature აღწერს თითოეული ტრანზაქციის სხვადასხვა მახასიათებელს: გადახდის დრო, თანხა, ბარათის ინფორმაცია, მოწყობილობის ინფორმაცია, ელ-ფოსტის დომენი და სხვა. target ცვლადი არის `isFraud`, ანუ ბინარული მნიშვნელობა, არის თუ არა ტრანზაქცია თაღლითური. მონაცემეში დიდი ინბალანსია, fraud-ის წილი მთლიან მონაცემებში მხოლოდ 3.5%-ია.

პროექტის მიზანი არ არის მხოლოდ საუკეთესო leaderboard ქულის მიღება. უფრო მნიშვნელოვანია, რომ სხვადასხვა მიდგომა გავტესტოთ cleaning-ის, feature engineering-ის, feature selection-ისა და მოდელირების ეტაპებზე, და გავიგოთ რა მუშაობს და რატომ. დავალების სპეციფიკიდან გამომდინარე, თითოეული არქიტექტურის საუკეთესო მოდელი შენახულია sklearn Pipeline-ად, რომელიც პირდაპირ ეშვება დაუმუშავებელ test set-ზე, ხოლო საუკეთესო მათგანი დარეგისტრირებულია Model Registry-ში.

## მიდგომა პრობლემის გადასაჭრელად

თავდაპირველად მონაცემებს შეუცვლელად დავაკვირდი, გავიგე რამდენიმე მნიშვნელოვანი ფაქტი მათ შესახებ რაც მათ გარდაქმნაში და დამუშავებაში დამეხმარებოდა. შემდგომ დავამუშავე ისინი რათა ყველა არარიცხვითი მნიშვნელობა რიცხვითად გადამექცია, ცარიელი ცვლადები შესაბამისი მნიშვნელობებით შემევსო. feature engineering-ის ნაწილში დავამატე რამდენიმე ცვლადი, რომელიც შესაძლოა მოდელისთვის მნიშვნელოვანი ყოფილიყო. გამოვიყენე რამდენიმე feature selection-ის ტექნიკა, და დავატრენინგე 3 სხვადასხვა არქიტექტურის მოდელი: Logistic Regression (წრფივი), Random Forest (bagging ensemble) და XGBoost (boosting ensemble). თითოეული მათგანისთვის ვცადე რამდენიმე კონფიგურაცია, ვნახე overfit/underfit შემთხვევები, შევადარე შედეგები და ამოვარჩიე საუკეთესო. საბოლოოდ საუკეთესო შედეგის მქონე XGBoost მოდელი დავარეგისტრირე MLflow Model Registry-ში სრული Pipeline-ის სახით.

## რეპოზიტორიის სტრუქტურა

* **`model-experiment-xgboost.ipynb`** XGBoost-ის notebook. შეიცავს EDA-ს, cleaning-ს, feature engineering-ს, feature selection-ს და XGBoost-ის ოთხი სხვადასხვა კონფიგურაციის ტრენინგს (baseline, overfit test, tuned, tuned + all features).
* **`model-experiment-xgboost-pipeline.ipynb`** XGBoost-ის საუკეთესო კონფიგურაცია, შეფუთული sklearn Pipeline-ად, რომელშიც preprocessing და მოდელი ერთ ობიექტადაა გაერთიანებული. ეს Pipeline დარეგისტრირებულია Model Registry-ში როგორც `ieee-fraud-best-model`.
* **`model-experiment-random-forest .ipynb`** Random Forest-ის notebook. შეიცავს სამი კონფიგურაციის ტრენინგს (baseline, overfit test, tuned) და ბოლოში დამატებულ Pipeline კოდს.
* **`model-experiment-logistic-regression .ipynb`** Logistic Regression-ის notebook. შეიცავს ოთხი კონფიგურაციის ტრენინგს (baseline, strong regularization, tuned, tuned + all features), Lasso რეგულარიზაციით feature selection-ს, და ბოლოში Pipeline-ს.
* **`model-inference.ipynb`** test set-ზე პროგნოზის გაკეთება. Model Registry-დან იტვირთება დარეგისტრირებული XGBoost Pipeline და გენერირდება `submission.csv` ფაილი, რომელიც Kaggle-ზე აიტვირთა.
* **`README.md`** მოცემული ფაილი, სადაც დეტალურად არის აღწერილი ყველა ეტაპი და მიდგომა.

## EDA

სანამ მონაცემებს შევცვლიდი, უბრალოდ დავაკვირდი მათ. transaction და identity ცხრილების შერწყმის შემდეგ, დატასეტს ჰქონდა 590540 სტრიქონი და 434 ცვლადი. 403 ცვლადი იყო რიცხვითი, 31 კატეგორიული. 414 ცვლადი (434-დან) შეიცავდა ცარიელ მნიშვნელობებს, ხოლო 214 ცვლადს ცარიელი მნიშვნელობების 50%-ზე მეტი ჰქონდა, რაც ნიშნავდა რომ ცარიელ მნიშვნელობებთან დაკავშირებით სერიოზული გადაწყვეტილებების მიღება დაგვჭირდებოდა. fraud-ის წილი იყო 3.5%, რაც ნიშნავს, რომ მონაცემები ინბალანსშია და ეს მოდელის ტრენინგისას უნდა გვახსოვდეს. ამ ყველაფერმა წარმოდგენა შემიქმნა, თუ რა სახის მონაცემებთან მქონდა საქმე.

## Data Separation

საწყისი მონაცემები 80/20 შეფარდებით გავყავი train set-ად და validation set-ად, `stratify=y` პარამეტრით, რათა ორივე ნაწილში fraud-ის 3.5%-იანი თანაფარდობა შენარჩუნებულიყო. სტრატიფიკაცია მნიშვნელოვანი იყო, რადგან 3.5%-იანი imbalanced კლასისას random split-ი შეიძლება ნაწილებში განსხვავებული პროცენტულობა მოეცა, რაც AUC-ის შეფასებას არაზუსტს გახდიდა.

## Feature Engineering

### Cleaning მიდგომები

ამ ნაწილში დავაკვირდი train set-ს და შევეცადე ცარიელი ცვლადების და კატეგორიული ცვლადების სწორად დამუშავებას. პირველ რიგში, დავიდროპე ის ცვლადები, რომლებსაც 90%-ზე მეტი ცარიელი მნიშვნელობა ჰქონდათ, რადგან ასეთი ცვლადებიდან რეალური სიგნალის ამოღება ფაქტობრივად შეუძლებელი იყო. ჯამში 12 ცვლადი დავიდროპე, რის შემდეგაც დარჩა 420 ცვლადი.

### Nan მნიშვნელობების დამუშავება

დარჩენილი ცარიელი მნიშვნელობები ბევრი იყო, რის გამოც მოვიფიქრე bulk მიდგომა: ყველა რიცხვითი ცარიელი მნიშვნელობა შევცვალე -999-ით, ხოლო ყველა კატეგორიული "missing" სტრინგით.

### კატეგორიული ცვლადების რიცხვითში გადაყვანა

შემდგომ კატეგორიული ცვლადები რიცხვითად გადავიყვანე Label Encoding-ით. სახლის ფასების დავალებისგან განსხვავებით, სადაც one-hot encoding გამოვიყენე, აქ ცვლადების რაოდენობა ისედაც ძალიან დიდი იყო (420), ამიტომ Label Encoding უფრო პრაქტიკული იყო.

### feature-ების დამატება

შემოვიღე 7 დამატებითი ცვლადი, რომელიც უკვე არსებული ცვლადების მანიპულაციით იყო მიღებული, მაგალითად ტრანზაქციის საათი, კვირის დღე, თანხის ლოგარითმი, გადარიცხული თანცის ათობითი ნაწილი და ა.შ. ამ ყველაფრის შემდეგ ცვლადების რაოდენობა 420-დან 427-მდე გაიზარდა.

## Feature Selection

ამ ეტაპზე ცვლადების რაოდენობაზე მანიპულაციისთვის გამოვიყენე რამდენიმე Feature Selection ალგორითმი. სხვადასხვა მოდელისთვის სხვადასხვა მიდგომა ავარჩიე:

* **XGBoost-ისთვის** გამოვიყენე **Tree-based Feature Importance**. პირველ რიგში დავატრენინგე სწრაფი XGBoost (100 trees, depth 5) ყველა feature-ზე, შემდეგ feature_importances_-დან ამოვიღე საუკეთესო 200 ცვლადი. ეს მიდგომა ბუნებრივად ერგება tree-based მოდელებს. შედეგად ვნახე, რომ V258 ცვლადი ძალიან მნიშვნელოვანი იყო (importance 0.247), ხოლო 140 ცვლადს ნულოვანი მნიშვნელობა ჰქონდა, ანუ XGBoost-ი მათ საერთოდ არ იყენებდა.

* **Random Forest-ისთვის** გამოვიყენე იგივე XGBoost-based Feature Importance და დავტოვე top 100 ცვლადი. 

* **Logistic Regression-ისთვის** გამოვიყენე **Lasso (L1 Regularization)** feature selection, რომელიც კარგად ერგება linear მოდელებს. C=0.01-ით ძლიერი რეგულარიზაციით 427 ცვლადიდან 261 ცვლადი დარჩა არანულოვანი კოეფიციენტით, დანარჩენი 166 ცვლადის კოეფიციენტი ნულამდე ჩამოვიდა.

რამდენიმე მეთოდის გამოყენების მიზეზი ისაა, რომ თითოეულ მათგანს თავისი bias აქვს და სხვადასხვა მოდელისთვის სხვადასხვა მუშაობს. ყველა selection მეთოდი გამოვიყენე **მხოლოდ** `X_train`-ზე, რათა validation set-ის ინფორმაციას არ ეთამაშა როლი feature-ების არჩევის პროცესში. ასევე, თითოეული მოდელისთვის გავუშვი ექსპერიმენტი **ყველა** feature-ით, რათა მენახა, რეალურად დაგვეხმარა Yუ არა feature selection-ი.

## Training

### ტესტირებული მოდელები

დავტესტე 3 არქიტექტურის მოდელი, თითოეული განსხვავებული მიზნით:

* **Logistic Regression** (L2 regularization) წრფივი მოდელი, რომელიც მარტივი და სწრაფი baseline-ია.
* **Random Forest** (bagging ensemble) არაწრფივი მოდელი, რომელიც ბევრი decision tree-ის გასაშუალოებით თავიდან იცილებს ერთი ხის overfitting-ს.
* **XGBoost** (boosting ensemble) gradient boosting მოდელი, რომელიც როგორც წესი საუკეთესო შედეგს იძლევა კლასიფიკაციის ამოცანებში.

თითოეული მოდელისთვის ვცადე რამდენიმე კონფიგურაცია (baseline, overfit test, tuned, ხოლო XGBoost-ისა და LogReg-ისთვის ასევე ყველა feature-ით), რომ overfit/underfit შემთხვევები მენახა და გამეგო, რომელი hyperparameter-ები რა ეფექტს ახდენდნენ ამოცანაზე. ყველა ექსპერიმენტი დავლოგე MLflow-ზე.

### Hyperparameter ოპტიმიზაციის მიდგომა

თითოეული მოდელისთვის გამოვცადე რამდენიმე კონფიგურაცია და გავაანალიზე შედეგები. იდეა იყო, რომ შესაძლო overfit-ის და underfit-ის შემთხვევებიც გამოჩენილიყო.

**XGBoost-ის შედეგები:**
* baseline (300 trees, depth 6): val_auc=0.9411, gap=+0.023
* overfit test (1000 trees, depth 12): train_auc=1.0000, val_auc=0.9720, gap=+0.028
* tuned (500 trees, depth 7, regularization, scale_pos_weight): val_auc=0.9506
* tuned + all features: val_auc=0.9497

**Random Forest-ის შედეგები:**
* baseline (depth=10): val_auc=0.8727, gap=+0.008
* overfit test (depth=None): train_auc=1.0000, val_auc=0.9218, gap=+0.078
* tuned (depth=15, balanced): val_auc=0.9156

**Logistic Regression-ის შედეგები:**
* baseline (C=1.0): val_auc=0.8306, gap=+0.001
* strong regularization (C=0.01): val_auc=0.8264, gap=-0.003 (underfit!)
* tuned (C=0.5, balanced): val_auc=0.8417
* tuned + all features: val_auc=0.8489

შედეგების ანალიზი რამდენიმე საინტერესო რამეს გვიჩვენებს:

1) **XGBoost იყო ყველა მოდელზე უკეთესი.** ეს ლოგიკურია, რადგან fraud detection-ში არსებობს ბევრი არაწრფივი კავშირი ცვლადებს შორის, რომელსაც boosting მოდელი კარგად აღიქვამს. LogReg ვერ ხედავდა ამ ურთიერთქმედებებს, ამიტომ AUC მხოლოდ 0.85 იყო.

2) **"overfit test"-მა საინტერესო შედეგი გამოიღო.** მართალია train AUC-მ მიაღწია 1.0-ს (ანუ მოდელმა training data სრულად დაიმახსოვრა), მაგრამ validation AUC მაინც მაღალი დარჩა (XGBoost-ზე 0.972, RF-ზე 0.922). ეს ნიშნავს, რომ ensemble-based მეთოდები (bagging + boosting + column subsampling) საკმაოდ კარგად ახდენენ overfitting-ის რეგულარიზაციას. ეს განსხვავებაა ერთი Decision Tree-სგან, რომელიც პირველ დავალებაში დიდი ოვერფიტი გვაჩვენა.

3) **strong regularization-ი LogReg-ზე underfit იყო.** C=0.01-ით val_auc იყო train_auc-ზე უფრო მაღალი, რაც ნიშნავს რომ მოდელი იმდენად შეზღუდული იყო, რომ training data-შიც ვერ ხედავდა სიგნალებს.

4) **feature selection-მა LogReg-ს მცირედად ავნო.** Lasso-ს მიერ შერჩეულ 261 ცვლადზე val_auc=0.8417, ხოლო ყველა 427 ცვლადზე val_auc=0.8489. ეს ნიშნავს, რომ Lasso-მ ზოგი სასარგებლო ცვლადი დაგვაკარგვინა.

### საბოლოო მოდელის შერჩევის დასაბუთება

ყველა შედეგის გათვალისწინებით, **XGBoost overfit_test კონფიგურაცია** აღმოჩნდა საუკეთესო (val_auc=0.9720). მიუხედავად "overfit_test"-ის სახელისა, ეს მოდელი რეალურად უფრო მეტ სიგნალს ხედავდა მონაცემებში, ვიდრე უფრო შეზღუდული ვერსიები. XGBoost-ის შიდა რეგულარიზაცია (column subsampling, learning rate shrinkage) საკმარისი იყო, რომ დიდი overfit-ი თავიდან აეცილებინა, მიუხედავად 1000 ხის რაოდენობისა და 12 სიღრმისა.

**Pipeline-ად შეფუთვა:** ცალკე notebook-ში (`model-experiment-xgboost-pipeline.ipynb`) გავაერთიანე ყველა preprocessing ნაბიჯი (cleaning, label encoding, feature engineering) ერთ custom transformer-ში (`FraudPreprocessor`). ამის შემდეგ, Pipeline-მა val_auc-ი 0.9750-მდე გააუმჯობესა (Pipeline-ი დატრენინგდა მთლიან raw data-ზე)

ეს სრული Pipeline დარეგისტრირდა Model Registry-ში როგორც `ieee-fraud-best-model`. სხვა ორი არქიტექტურისთვის (Random Forest და Logistic Regression) Pipeline ვერსიები ასევე შენახულია MLflow-ზე, მაგრამ Model Registry-ში არ არის რეგისტრირებული, რადგან XGBoost-მა გაცილებით უკეთეს შედეგი აჩვენა.

## MLflow Tracking

### MLflow ექსპერიმენტების ბმული

[https://dagshub.com/llikl23/IEEE-CIS-Fraud-Detection.mlflow](https://dagshub.com/llikl23/IEEE-CIS-Fraud-Detection.mlflow)

სამი ცალკე ექსპერიმენტი შევქმენი თითოეული არქიტექტურისთვის:
* **`XGBoost_Training`** XGBoost-ის ყველა run (კონფიგურაციები + FINAL_XGBoost_best + XGBoost_Pipeline_FINAL).
* **`RandomForest_Training`** RF-ის ყველა run (კონფიგურაციები + FINAL_RF_best + RF_Pipeline_FINAL).
* **`LogisticRegression_Training`** LogReg-ის ყველა run (კონფიგურაციები + FINAL_LogReg_best + LogReg_Pipeline_FINAL).

### ჩაწერილი მეტრიკების აღწერა

თითოეული run-ისთვის MLflow-ში დავლოგე შემდეგი ინფორმაცია:

* **Parameters:** მოდელის ტიპი, hyperparameter-ები (n_estimators, max_depth, learning_rate, C, regularization, class_weight და სხვა), feature-ების რაოდენობა.
* **Metrics:** `train_auc`, `val_auc` (ორივე AUC-ROC-ის სახით, რაც ემთხვევა Kaggle-ის მეტრიკას), და `overfit_gap` (train_auc - val_auc, რომელიც overfitting-ის ნახვისთვის გამოვიყენე).
* **Artifacts:** მოდელის სრული Pipeline (preprocessor + scaler + model), რომელიც პირდაპირ ეშვება დაუმუშავებელ test set-ზე.

### საუკეთესო მოდელის შედეგები

Architecture:    XGBoost (Pipeline-ად შეფუთული)

Configuration:   1000 trees, max_depth=12, learning_rate=0.1

Pipeline:        FraudPreprocessor → XGBClassifier

Val AUC:         0.9750

Kaggle public:   0.931836

Kaggle private:  0.896036
