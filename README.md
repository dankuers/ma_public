# Hinweis
Marie-Sophie hat mich gebeten den Code rauszunehmen und privat zu machen. Daher befindet sich dieser ab jetzt in dem hier hochgeladenen Ziparchiv. Das Passwort zum Entpacken ist meine Matrikelnummer, welche sich auf dem Deckblatt meiner Masterarbeit befindet.

# General notes
Note that most settings were kept the same as in the tissue outcome prediction code that was provided to me.

The model weights were not uploaded because they were created using data that is not publicly available. The settings to create these models can be found in the appendix of the thesis. The models weights might be shared upon request.

Note that some features implemented were not used (or only tested out of curiosity) in the thesis, e.g. logarithmic transformations to alter (unskew) distributions or the rudimentary form of data augmentation.

# Implementation notes / how to use
Data is ingested using an object created with dataset.py. This dataset object is created using a parameter object, which is instantiated using params.py.
First, instantiate the parameter object, then change the preset/default parameters in that object (no need to actually change the params.py).
After that, call set_split in the params object.
Then pass the params object to the dataset constructor for every type of dataset, i.e. training, validation and test.

This will return an enriched pytorch Dataset object that can be used with a pytorch Dataloader. 
Apart from being a pytorch Dataset object, this object also features access to the ingested data, which are pandas DataFrames which can be inspected using:
- `obj.X` for the possible features (note that not all columns have to be used, this is defined later in the `__getitem__` function)
- `obj.y` for the target data
- `obj.data` for all the raw ingested data

Note that you have to implement the `__init__` and `__getitem__` function of the Dataset, an example can be found in model_implementation_example/dataset_params.py.
If you only want to look at the data you can leave `__getitem__` empty. `__getitem__` is what is needed for the DataLoader.

To actually use this Dataset object during training, create a Dataloader out of it as in model_implementation_example/train.py. If you're using imaging data you will also have to call a transformation function in train.py. This is because imaging data is not loaded to memory upon creation of the dataset object, only the path is loaded. The Dataset object will look in a predefined path if the transformation is already saved on the disk. If yes, then it will simply load that Tensor file (speeds up the pre-training process enormously), otherwise it will load the imaging file from disk and apply the transformations as defined in the parameter file.

For convenience, one can also define a get_params function as in model_implementation_example/dataset_params.py, which only needs to be called in model_implementation_example/train.py. In that way, the train.py file will not be filled with parameter settings and makes it easier to save the parameters used (e.g. by simply copying the contents of dataset_params.py, assuming that the params.py default settings won't be changed).

The file model_implementation_example/model.py is simply an outsourcing of the architecture of the model.

The runs folder is used for checkpointing and saving results.

# How to transform and save the imaging data
I.e. avoiding the need to compute transformations every time training is commenced, resulting in much faster loading times.

In every patient's imaging folder, e.g. data/CT/564/ for patient with patient_id 564, create a folder called precomputed, i.e. data/CT/564/precomputed.

Then setup the training process until the step where the imaging data is transformed, i.e. after the creation of the Dataset objects. For each train, val and test-dataset-object call obj.transform_all_modalities_all_patients_and_save_tensor. This will transform the imaging modalities as defined in the params object and save the resulting Tensor in the precomputed folder.

To use these precomputed Tensors, simply call the transformation function of your choice, e.g. transform_modality_all_patients. If a precomputed Tensor exists, then it will loaded without the need of transforming the imaging data again.
