# nlp-b2w-reviews

## Instale as dependências

```bash
pip install -r requirements.txt
```

## Inicialize o `mlflow server`

```bash
mlflow server -p 8889 -h 0.0.0.0
```

## Passos para gerar a pipeline

```bash
mkdir -p pipeline

dvc init

dvc run -o nlp_b2w_reviews/__init__.py -f pipeline/setup.dvc \
    "nbdev_build_lib"

dvc run -o data/B2W-Reviews01.csv -f pipeline/download_dataset.dvc \
    "wget -P data https://github.com/b2wdigital/b2w-reviews01/raw/master/B2W-Reviews01.csv"

dvc run -d nlp_b2w_reviews/__init__.py -d notebooks/00_preprocess.ipynb -o nlp_b2w_reviews/preprocess.py -f pipeline/build_preprocess.dvc \
    "nbdev_build_lib --fname notebooks/00_preprocess.ipynb"

dvc run -d nlp_b2w_reviews/preprocess.py -d data/B2W-Reviews01.csv -o data/normalized_reviews.csv -f pipeline/run_preprocess.dvc \
    "python nlp_b2w_reviews/preprocess.py"

dvc run -d nlp_b2w_reviews/__init__.py -d notebooks/01_dataset_splitter.ipynb -o nlp_b2w_reviews/dataset_splitter.py -f pipeline/build_dataset_splitter.dvc \
    "nbdev_build_lib --fname notebooks/01_dataset_splitter.ipynb"

dvc run -d nlp_b2w_reviews/dataset_splitter.py -d data/normalized_reviews.csv -o data/train.csv -o data/test.csv -f pipeline/run_dataset_splitter.dvc \
    "python nlp_b2w_reviews/dataset_splitter.py"

dvc run -d nlp_b2w_reviews/__init__.py -d notebooks/02_trainer.ipynb -o nlp_b2w_reviews/trainer.py -f pipeline/build_trainer.dvc \
    "nbdev_build_lib --fname notebooks/02_trainer.ipynb"

dvc run -d nlp_b2w_reviews/trainer.py -d data/train.csv -o models/sentiment_analyzer.joblib -f pipeline/run_trainer.dvc \
    "mkdir -p models && python nlp_b2w_reviews/trainer.py"

dvc run -d nlp_b2w_reviews/__init__.py -d notebooks/03_evaluator.ipynb -o nlp_b2w_reviews/evaluator.py -f pipeline/build_evaluator.dvc \
    "nbdev_build_lib --fname notebooks/03_evaluator.ipynb"

dvc run -d nlp_b2w_reviews/evaluator.py -d data/test.csv -d models/sentiment_analyzer.joblib -o mlruns -f pipeline/run_evaluator.dvc \
    "python nlp_b2w_reviews/evaluator.py"

dvc run -d mlruns -f Dvcfile \
    "ls pipeline"
```

## Visualize a pipeline

```bash
dvc pipeline show --ascii
```
