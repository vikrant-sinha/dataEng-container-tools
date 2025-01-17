# Data Engineering Container Tools

This packages is split into five parts: **CLA**, **GCS**, **safe_stdout**, **DB** and **simple_setup**.

## CLA:
Deals with receiving input from the command line. Has three classes: `custom_command_line_argument`, `command_line_argument_type`, and `command_line_arguments`.

* `custom_command_line_arguments`: Acts as a container for custom command line arguments. All of the attributes available when creating command line arguments through [the `parser.add_argument()` method](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser.add_argument) from [the `argparse` library](https://docs.python.org/3/library/argparse.html) are available in this class. Has the following methods:
  * `__init__`: Creates the class with the following inputs:
    * `name`: Required. The name of the command line argument. Should be given without the preceding '--', which will be added automatically.
    * `action`: Optional. Defaults to `None`. The basic type of action to be taken when this argument is encountered at the command line.
    * `nargs`: Optional. Defaults to `None`. The number of command-line arguments that should be consumed. A number, `'*'`, or `'+'`.
    * `const`: Optional. Defaults to `None`. A constant value required by some action and nargs selections.
    * `default`: Optional. Defaults to `None`. The value produced if the argument is absent from the command line.
    * `data_type`: Optional. Defaults to `None`. The type to which the command-line argument should be converted.
    * `choices`: Optional. Defaults to `None`. A container of the allowable values for the argument.
    * `required`: Optional. Defaults to `None`. Whether or not the command-line option may be omitted (optionals only).
    * `help_message`: Optional. Defaults to `None`. A brief description of what the argument does.
    * `metavar`: Optional. Defaults to `None`. A name for the argument in usage messages.
    * `dest`: Optional. Defaults to `None`. The name of the attribute to be added to the object returned by parse_args().
  
* `command_line_argument_type`: Enumeration type. Used for populating initialization fields in `command_line_arguments`. Has the following types:
  * `OPTIONAL`: Indicates the associated command line argument should be created as optional.
  * `REQUIRED`: Indicates the associated command line argument should be created as required.
  
* `command_line_arguments`: Creates and parses command line arguments. Includes helper functions for using the command line inputs. Has the following methods:
  * `__init__`: Created the class with the following inputs:
    * `input_files`: Optional `command_line_argument_type`. Defaults to `None`. If `REQUIRED`, will add `--input_bucket_names`, `--input_paths`, and `--input_filenames` as required command line inputs. If `OPTIONAL`, will add them as optional. If `None` they will not be added.
    * `output_files`: Optional `command_line_argument_type`. Defaults to `None`. If `REQUIRED`, will add `--output_bucket_names`, `--output_paths`, and `--output_filenames` as required command line inputs. If `command_line_argument_type.OPTIONAL`, will add them as optional. If `None` they will not be added.
    * `secret_locations`: Optional `command_line_argument_type`. Defaults to `None`. If `REQUIRED`, will add `--secret_locations` as required command line input. If `OPTIONAL`, will add it as optional. If `None`, it will not be added.
    * `default_file_type`: Optional `command_line_argument_type`. Defaults to `None`. If `REQUIRED`, will add `--default_file_type` as required command line argument. If `OPTIONAL`, will add it as optional. Input can be one of `parquet`, `csv`, `pkl`, or `json`, with the default being `parquet`. If `None`, the command line argument will not be added. 
    * `custom_inputs`: Optional list of `custom_command_line_arguments`. Defaults to `None`. All items in list will be added to the command line arguments.
    * `description` : Optional `string`. Defaults to `None`. A description to be printed when the command line argument `--help` is used.
    * `input_dtypes`: Optional `command_line_argument_type`. Defaults to `None`. If `input_files` is `None`, then this does nothing. If `input_files` is not `None` and `input_dtypes` is `REQUIRED`, will add `--input_dtypes`as a required command line input. If `OPTIONAL`, will add it as optional.  Input is a JSON dictionary of (column: type) pairs.
    * `parser`: Optional `argparse.ArgumentParser`. Defaults to `None`. A parser on which to add the command line arguments and parse. If `None` one will be created.
    * `running_local`: Optional `argparse.ArgumentParser`. Defaults to `None`. A flag for determining whether or not the script is running locally. Defaults to False.
  * `get_arguments`: Returns the arguments passed in through the command line as a `Namespace` object.
  * `get_input_dtypes`: Returns the input dtypes passed in through the command line.
  * `get_input_uris`: Returns the input_uris passed in through the command line as a list of strings. All of the format `gs://[BUCKETNAME]/[FILEPATH]/[FILENAME]`. If one bucket is specified, the same bucket is used for every file path. If more than one bucket is specified, one bucket is used for one file path, and there must be a 1:1:1 ratio of buckets to filepaths, to file names.
  * `get_output_uris`: Returns the output_uris passed in through the command line as a list of strings. All of the format `gs://[BUCKETNAME]/[FILEPATH]/[FILENAME]`. If one bucket is specified, the same bucket is used for every file path. If more than one bucket is specified, one bucket is used for one file path, and there must be a 1:1:1 ratio of buckets to filepaths, to file names.
  * `get_secret_locations`: Returns the secret locations passed in through the command line as a list of strings.
  * `get_secrets`: Returns a dictionary of objects. The key is the file name of the secret, and the object is that file loaded using 'json.load()`.
  * `check_args`: Does nothing. In future this will error check the arguments passed in through the command line.
  
## GCS:
Deals with uploading and downloading files to/from GCS. Has one class `gcs_file_io` with the following methods:
* `__init__`: Creates the class with the following inputs:
  * `gcs_secret_location`: Required. The location of the secret file needed for GCS.
  * `local`: Optional. Defaults to `False`. If `True`, no contact will be made with GCS.
* `download_file_to_object`: Downloads a file from GCS to an object in memory:
  * `gcs_uri`: Required. The uri of the object in GCS to download. If `local` is `True`, it is the path to a local file that will be read into an object.
  * `default_file_type`: Optional. Defaults to `None`. If the uri the object does not have a file type ending, it will be assumed to be this type.
  * `dtype`: Optional. Defaults to `None`. A dictionary of (column: type) pairs.
  * `header`: Optional, Default to `0`. If set to `None` it will not read first row as header, only for xls and csv files, if set to `0` or any `int` or `List[int]` it will read those rows to build header/columns
* `download_files_to_objects`: Downloads files from GCS to objects in memory:
  * `gcs_uris`: Required. The uris of the object in GCS to download. If `local` is `True`, it is the paths to local files that will be read into objects.
  * `default_file_type`: Optional. Defaults to `None`. A string. If the uri an object does not have a file type ending, it will be assumed to be this type.
  * `dtypes`: Optional. Defaults to empty list. A list of dictionary of (column: type) pairs.
  * `headers`: Optional. Default to empty list. A list of headers of the file
* `download_file_to_disk`: Downloads a file from GCS to the container's hard drive:
  * `gcs_uri`: Required. The uri of the object in GCS to download. If `local` is `True`, it is the path to a local file that will be copied to `local_location`.
  * `local_location`: Optional. Defaults to `None`. Where to save the object. If `None`, saves to same path as the the GCS URI.
* `download_files_to_disk`: Downloads files from GCS to the container's hard drive:
  * `gcs_uris`: Required. The uris of the objects in GCS to download. If `local` is `True`, it is the paths to local files that will be copied to `local_locations`.
  * `local_locations`: Optional. Defaults to empty list. The locations to save the objects. If empty, saves to same paths as the the GCS URIs.
* `upload_file_from_object`: Uploads a file to GCS from an object in memory:
  * `gcs_uri`: Required. The uri to which the object will be uploaded. If `local` is `True`, it is the path to a local file where the object will be written.
  * `default_file_type`: Optional. Defaults to `None`. If the uri does not have a file type ending, it will be assumed to be this type.
  * `header`: Optional. Defaults to `True`, Write out the column names (for csv and excel)
  * `index`: Optional. Default to False, Whether to write the index or not (for csv and excel)
  * `dtype`: Optional. Defaults to `None`. A dictionary of (column: type) pairs.
  * `metadata`: Optional dictionary. Defaults to an empty dictionary. The metadata to add to the object. Git hash is added automatically if `GITHUB_SHA` is set as an enviornment variable.
* `upload_files_from_objects`: Uploads files to GCS from objects in memory:
  * `gcs_uris`: Required. The uris to which the objects will be uploaded. If `local` is `True`, it is the paths to local files where the objects will be written.
  * `default_file_type`: Optional. Defaults to `None`. A sting. If the uri an object does not have a file type ending, it will be assumed to be this type.
  * `dtypes`: Optional. Defaults to `None`. A list of dictionary of (column: type) pairs.
  * `headers`: Optional , Default to `[]`. Only for `csv` and `xls` files, list of boolean value for each object , if length of `headers` is `1` then `headers[0]` will be used while writing all object, if `length` is greater than `1` then for each `ith` object `ith` header will be passed , else default value `True` will be passed. `header` value controls whether we want to  write header of dataframe or not
  * `indices` : Optional. Defaults to [] , List of boolean value for index (if index is True then index will be written)
* `upload_file_from_disk`: Uploads a file to GCS from the container's hard drive:
  * `gcs_uri`: Required. The uri to which the object will be uploaded. If `local` is `True`, it is the path to a local file that will be copied from `local_location`.
  * `local_location`: Optional. Defaults to `None`. The location of the object. If `None`, assumes the same path as the the GCS URI.
  * `metadata`: Optional dictionary. Defaults to an empty dictionary. The metadata to add to the object. Git hash is added automatically if `GITHUB_SHA` is set as an enviornment variable.
* `upload_files_from_disk`: Uploads files to GCS from the container's hard drive:
  * `gcs_uris`: Required. The uris to which the objects will be uploaded. If `local` is `True`, it is the paths to local files that will be copied from `local_locations`.
  * `local_locations`: Optional. Defaults to `None`. The locations of the objects. If `None`, assumes the same paths as the the GCS URIs.
  * `metadata`: Optional list of dictionaries. Defaults to empty. The metadata to add to the objects. Git hash is added automatically if `GITHUB_SHA` is set as an enviornment variable.

## DB:
Deals with datastore operations. Has one method `get_secrets` and one class `Db`. The following is the discription:
* `get_secrets`: get secrets from vault mounted json file:
  * `path_`: Required. path to credentials file
* `__init__`: Creates the class with the following inputs:
  * `task_kind`: Required. The kind of the task for which datastore operations is being performed.
* `get_data_store_client`: creates and return datastore client:
  * `PATH`: Required. path to credentials file
* `get_task_entry`: static method. Used to query the entry for task. Returns a list of the entry:
  * `kind`: Required. kind to query on
  * `filter_map`: Required. filter map (dictionary)
  * `client`: Required. data store client
  * `order_task_entries_params`: Optional. json object containing below two key-value pairs
    * `order_by_key_list`- list of parameters to order the task entries
    * `descending_order`- True/False
* `put_snapshot_task_entry`: Stores the entry for the task:
  * `client`: Required. datastore client
  * `task_entry`: Required. Entity which store actual instance of data
  * `params`: Required. dictionary containing all the parameters(key-value pairs) to be stored
* `handle_task`: it's used to check if the task instance for the given param is available or not.
        If task instance is already present then it will update the existing instance else
        create a new instance and store it to given Entity.:
  * `client`: Required. datastore client
  * `params`: Required. dictionary containing all the parameters(key-value pairs) to be stored
  * `order_task_entries_params`: Optional. parameters to order the task entries if required


## safe_stdout:
Ensures that secrets are not accidentally printed using stdout. Has one class `safe_stdout`, two helper methods, `setup_stdout` and `setup_default_stdout`, and one global variable `default_secret_folder`:

* `safe_stdout`: The output class in charge of outputting to the command line. Replaces stdout. Has the following methods:
  * `__init__`: Creates the class with the following inputs:
    * `bad_words`: Required. A list of words to censor from output.
  * `write`: Writes a message to the command line. Usually called through Python's built in `print()` function. Has the following inputs:
    * `message`: Required. The message to write.
  * `add_words`: Adds a list of words to the list of words being censored. Has the following inputs:
    * `bad_words`: Required. A list of wors to censor from output.
  
* `setup_stdout`: Censors all the values in a list of secret files from stdout. Takes the following input:
  * `secret_locations`: Required. A list of secret file locations.
  
* `setup_default_stdout`: Censors all values from secret files contained in folder. Takes the following input:
  * `folder`: Optional. Defaults to `default_secret_folder`. The path of the folder containing the secret files.
  
* `default_secret_folder`: Variable containing the folder in which secrets are injected by default. Currently set to `'/vault/secrets/'`.

## Example:
An example workflow using the classes above might look something like this:
```
from dataEng_container_tools.cla import command_line_arguments, command_line_argument_type
from dataEng_container_tools.gcs import gcs_file_io

my_inputs = command_line_arguments(secret_locations=command_line_argument_type.OPTIONAL,
                                   input_files=command_line_argument_type.REQUIRED,
                                   output_files=command_line_argument_type.REQUIRED)

input_uris = my_inputs.get_input_uris()
output_uris = my_inputs.get_output_uris()
secret_locations = my_inputs.get_secret_locations()                              
file_io = gcs_file_io(gcs_secret_location = secret_locations[0])
pqt_obj = file_io.download_file_to_object(input_uris[0])
#
# Edit the object in some way here.
#
result = file_io.upload_file_from_object(gcs_uri=output_uris[0], object_to_upload=pqt_obj)
```

## simple_setup:
A simple way to get input from the command line, and download and upload documents to/from GCS. Fewer options than the classes above but also fewer lines of code to write. A brief example (documentation to come):
```
from dataEng_container_tools.simple_setup import simple_setup
simple = simple_setup(['input_left', 'input_right', 'output_inner', 'output_outer', 'secret_location', 'example_flag'])
objects = simple.get_input_objects()
input_left_object = objects['input_left']
input_right_object = objects['input_right']
#
# Edit the objects in some way here.
#

return_objs = {'output_outer': output_outer_object, 'output_inner': output_inner_object}
upload = simple.upload_objects(return_objs)
```
