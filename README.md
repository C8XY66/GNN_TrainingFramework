# **Graph Neural Network Training Framework**

This project contains a Python-based implementation of a training framework for Graph Neural Networks (GNNs). 
Specifically implemented are the Graph Isomorphism Network (GIN) as well as the Deep Graph Convolutional Neural Network (DGCNN).

The goal of the experiment was to investigate the effectiveness of GNNs in exploiting graph structure for graph classification tasks.

The performance of the models can be examined on graphs with node features and on graphs with their node features removed, leaving only graph structural information. 
Additionally, the models can be compared to a baseline MLP model, that only uses node features to classify the graphs.

## Further Reading

The model architecture is described in the original papers:
* GIN: ["How Powerful are Graph Neural Networks?" by Xu et al.](https://arxiv.org/abs/1810.00826) 
* DGCNN: ["An End-to-End Deep Learning Architecture for Graph Classification" by Zhang et al. ](https://ojs.aaai.org/index.php/AAAI/article/view/11782) 

A key reference point for the experiment and basis for the evaluation framework is provided by Errica et al.:
* ["A Fair Comparison of Graph Neural Networks for Graph Classification" by Errica et al.](https://arxiv.org/abs/1912.09893) 


## Requirements

This framework uses the following machine learning libraries:

* **PyTorch Geometric (PyG):** An extension library for PyTorch that provides implemented GNN architectures and methods, as well as common benchmark datasets and tools for handling graph data. 
* **PyTorch Lightning:** A wrapper for PyTorch that simplifies complex training pipelines by abstracting boilerplate code, thus making it more readable. It also manages training loops and data handling.
* **Optuna:** A library designed for hyperparameter optimisation which provides an interface for managing and tracking optimisation trials.


The libraries can be installed via pip:

```
pip install torch-geometric torch-lightning optuna 
```

## User Inputs

The script accepts various command line arguments for customisation of the experiment. You can provide these inputs as follows:
```
python script_name.py --EXPERIMENT WithNF --MODEL GIN --DATASET MUTAG --MAIN_DIR /Users/"username"/
```
Explanation of the arguments:

* --MAIN_DIR: Main directory of the project, where logs are saved.
* --EXPERIMENT: Type of experiment: 'WithNF', 'WithoutNF' (default: 'WithNF'). Specifies whether the graphs will include node features or not.
* --MODEL: Name of the model: 'GIN', 'DGCNN', 'MLP' (default: 'GIN'). Determines the GNN model to be used.
* --DATASET: Name of the dataset: 'MUTAG', 'NCI1', 'PROTEINS', 'IMDB-BINARY', 'IMDB-MULTI' (default: 'MUTAG'). Specifies the dataset to be used.
* --FOLDS: Number of folds the dataset is split into (default: 10).
* --REPS: Number of total repetitions (default: 10).
* --EPOCHS: Number of epochs to train each trial of a fold (default: 1000).
* --PATIENCE: Patience for early stopping (default: 100).
* --TRIALS: Number of trials for hyperparameter optimisation (default:50).
* --START_REP: From which repetition to start (default: 0).
* --STOP_REP: At which repetition to stop (default: None).
* --START_FOLD: From which fold to start (default: 0).
* --STOP_FOLD: At which fold to stop (default: None).
* --PARENT_DIR: Name of parent directory for resuming an interrupted run (default: None).
* --RUNNING_MODE: Running mode: 'single', 'parallel' (default: 'single'). Code executed as single run or folds run separately in parallel.

The script checks the validity of these inputs before running the experiment.


## Code Structure

The GNN training framework consists of the following major components:

* **GNN Models (GINModel, DGCNNModel, MLPModel):** 
These are implementations of the GIN, DGCNN, and MLP model for graph classification tasks. They are subclasses of PyTorch's nn.Module.
* **LightningModule (GNNModule):** 
This is a PyTorch Lightning wrapper for the GNN models. It defines the forward function, the loss function, the optimizer, and the training/ validation/ test steps.
* **LightningDataModule (GraphDataModule):** 
This PyTorch Lightning class manages the data loading and processing pipeline. It handles operations such as data preparation, setup for different stages, and it defines the data loaders.
* **Optuna Study and Trial Objects:** The Study optimises hyperparameters based on an objective function. Every Trial is one execution of the objective function, which returns the validation accuracy for one hyperparameter configuration.

* **Trainer:** A PyTorch Lightning trainer with early stopping, trial pruning and model checkpoint callbacks as well as a TensorBoardLogger.
* **SQLite Database:** The final test accuracies are saved to a SQLite Database with a lock preventing simultaneous write operations.

Please refer to the code comments and the official documentation of PyTorch Geometric, PyTorch Lightning, and Optuna for more details about these components.

## Evaluation Framework

For the evaluation framework, a 10 time repetition of a 10-fold cross validation is used. 
For the model selection, an inner holdout technique with a 90% / 10% split is used. Once the best model is selected, it is directly tested on the test set. 
All data splits are stratified.


