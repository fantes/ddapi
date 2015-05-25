---
title: API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='#'>http://www.deepdetect.com/</a>
  - <a href='http://github.com/beniz/deepdetect'>Documentation Powered by Slate</a>

includes:
  - connectors
  - errors

search: true
---

# Introduction

Welcome to the DeepDetect API!

TODO: main principles:
- server
- services and input / output connectors
- ML libraries and their options
- train / predict

# Info

## Get Server Information

```shell
curl -X GET "http://localhost:8080/info"
```

> The above command returns JSON of the form:

```json
{
	"status":{
		"code":200,
		"msg":"OK"
		},
	"head":
		{
		"method":"/info",
		"version":"0.1",
		"branch":"master",
		"commit":"e8592d5de7f274a82d574025b5a2b647973fccb3",
		"services":[]
		}
}
```

Returns general information about the deepdetect server, including the list of existing services.

### HTTP Request

`GET /info`

### Query Parameters

None

# Services

Create, get information and delete machine learning services

## Create a service

> Create a service from a multilayer Neural Network template, taking input from a CSV for prediction over 9 classes with 3 layers.

``` shell
curl -X PUT "http://localhost:8080/services/myserv" -d "{\"mllib\":\"caffe\",\"description\":\"example classification service\",\"type\":\"supervised\",\"parameters\":{\"input\":{\"connector\":\"csv\"},\"mllib\":{\"template\":\"mlp\",\"nclasses\":9,\"layers\":[512,512,512],\"activation\":\"PReLU\"}},\"model\":{\"repository\":\"/home/me/models/example\"}}"
```

> If "/home/me/models/example" correctly exists, the output is

```json
{"status":{"code":201,"msg":"Created"}}
```

Creates a new machine learning service on the server.

### HTTP Request

`PUT /services/<service_name>`

### Query Parameters

#### General

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
mllib | string | No | N/A  | Name of the Machine Learning library (e.g. 'caffe')
type | string | No | "supervised" | ML type "supervised"
description | string | yes | empty | Service description
model | object | No | N/A | Information for the statistical model to be built and/or used by the service
input | object | No | N/A | Input information for connecting to data
output | object | yes | empty | Output information

- Model Object

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
repository | string | No | N/A | Repository for the statistical model files
templates | string | yes | templates | Repository for model templates


#### Connectors

- Input Object

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
connector | string | No | N/A | Either "image" or "csv", defines the input data format
width | int | yes | 227 | Resize images to width ("image" only)
height | int | yes | 227 | Resize images to height ("image" only)
bw | bool | yes | false | Treat images as black & white

TODO: see section on Connectors

- Output object

TODO: best, etc...

#### Machine learning libraries

- Caffe

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
gpu | bool | yes | false | Whether to use GPU
gpuid | int | yes | 0 | GPU id
nclasses | int | yes | auto | Number of output classes ("supervised" service type)
template | string | yes | empty | Neural network template, from "lregression", "mlp", "alexnet", "googlenet", "ninnet"
layers | array of int | yes | [50] | Number of neurons per layer ("mlp" only)
activation | string | yes | relu | Unit activation ("mlp" only), from "sigmoid","tanh","relu","prelu"
dropout | real | yes | 0.5 | Dropout rate between layers ("mlp" only)

TODO: see Caffe and templates section

## Get information on a service

```shell
curl -X GET "http://localhost:8080/services/myserv"
```

> Assuming the service 'myserv' was previously created, yields

```json
{
  "status":{
	     "code":200,
	     "msg":"OK"
	  },
  "body":{
	     "mllib":"caffe",
	     "description":"example classification service",
	     "name":"myserv",
	     "jobs":
		{
		  "job":1,
		  "status":"running"
		}
	 }
}
```

Returns information on an existing service

### HTTP Request

`GET /services/myserv`

### Query Parameters

None

## Delete a service

```shell
curl -X DELETE "http://localhost:8080/services/myserv?clear=full"
```

> Yields

```json
{"status":{"code":200,"msg":"OK"}}

```

### HTTP Request

`DELETE /services/myserv`

### Query Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
clear | string | yes | mem | "full", "lib" or "mem". "full" clears the model and service repository, "lib" removes model files only according to the behavior specified by the service's ML library, "mem' removes the service from memory without affecting the files

# Train

Trains a statistical model from a dataset, the model can be further used for prediction

TODO: blocking vs asynchronous jobs

## Launch a training job

> Blocking train call from CSV dataset

```shell
curl -X POST "http://127.0.0.1:8080/train" -d "{\"service\":\"myserv\",\"async\":false,\"parameters\":{\"mllib\":{\"gpu\":true,\"solver\":{\"iterations\":300,\"test_interval\":100},\"net\":{\"batch_size\":5000}},\"input\":{\"label\":\"target\",\"id\":\"id\",\"separator\":\",\",\"shuffle\":true,\"test_split\":0.15,\"scale\":true},\"output\":{\"measure\":[\"acc\",\"mcll\"]}},\"data\":[\"/home/me/example/train.csv\"]}"
```
```json
{"status":{"code":201,"msg":"Created"},"body":{"measure":{"iteration":299.0,"train_loss":0.6463099718093872,"mcll":0.5919793284503224,"acc":0.7675070028011205}},"head":{"method":"/train","time":403.0}}

```

> Asynchronous train call from CSV dataset


```shell
curl -X POST "http://127.0.0.1:8080/train" -d "{\"service\":\"myserv\",\"async\":true,\"parameters\":{\"mllib\":{\"gpu\":true,\"solver\":{\"iterations\":100000,\"test_interval\":1000},\"net\":{\"batch_size\":512}},\"input\":{\"label\":\"target\",\"id\":\"id\",\"separator\":\",\",\"shuffle\":true,\"test_split\":0.15,\"scale\":true},\"output\":{\"measure\":[\"acc\",\"mcll\"]}},\"data\":[\"/home/me/models/example/train.csv\"]}"
```
```json
{"status":{"code":201,"msg":"Created"},"head":{"method":"/train","job":1,"status":"running"}}
```

> Requesting the status of an asynchronous training job:

```shell
curl -X GET "http://localhost:8080/train?service=myserv&job=1"
```
```json
{"status":{"code":200,"msg":"OK"},"head":{"method":"/train","job":1,"status":"running","time":74.0},"body":{"measure":{"iteration":445.0,"train_loss":0.7159726023674011,"mcll":2.1306082640485237,"acc":0.16127989657401424}}}
```

Launches a blocking or asynchronous training job from a service

### HTTP Request

`PUT or POST /train`

### Query Parameters

#### General

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
service | string | No | N/A | service resource identifier
async | bool | No | true | whether to start a non-blocking training call
data | object | yes | empty | input dataset for training, in some cases can be handled by the input connectors, in general non optional though

#### Input Connectors

- Image

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
width | int | yes | 227 | Resize images to width ("image" only)
height | int | yes | 227 | Resize images to height ("image" only)
bw | bool | yes | false | Treat images as black & white
test_split | real | yes | 0 | Test split part of the dataset
shuffle | bool | yes | false | Whether to shuffle the training set (prior to splitting)

- CSV

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
label | string | no | N/A | Label column name
ignore | array of string | yes | empty | Array of column names to ignore
label_offset | int | yes | 0 | Negative offset (e.g. -1) s othat labels range from 0 onward
separator | string | yes | ',' | Column separator character
id | string | yes | empty | Column name of the training examples identifier field, if any
scale | bool | yes | false | Whether to scale all values into [0,1]
test_split | real | yes | 0 | Test split part of the dataset
shuffle | bool | yes | false | Whether to shuffle the training set (prior to splitting)

#### Machine learning libraries

- Caffe

General:

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
gpu | bool | yes | false | Whether to use GPU
gpuid | int | yes | 0 | GPU id to use

Solver:

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
iterations | int | yes | N/A | Max number of solver's iterations
snapshot | int | yes | N/A | Iterations between model snapshots
snapshot_prefix | string | yes | empty | Prefix to snapshot file, supports repository
solver_type | string | yes | SGD | from "SGD", "ADAGRAD" and "NESTEROV"
test_interval | int | yes | N/A | Number of iterations between testing phases
test_initialization | bool | false | N/A | Whether to start training by testing the network
lr_policy | string | yes | N/A | learning rate policy ("steps", "inv", "fixed", ...)
base_lr | real | yes | N/A | Initial learning rate
gamma | real | yes | N/A | Learning rate drop factor
stepsize | int | yes | N/A | Number of iterations between the dropping of the learning rate
momentum | real | yes | N/A | Learning momentum
power | real | yes | N/A | Power applicable to some learning rate policies

Note: most of the default values for the parameters above are to be found in the Caffe files describing a given neural network architecture, or within Caffe library, therefore regarded as N/A at DeepDetect level.

Net:

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
batch_size | int | yes | N/A | Training batch size

TODO: note on batch_size, test batch_size ?

## Get information on a training job

> Requesting the status of an asynchronous training job:

```shell
curl -X GET "http://localhost:8080/train?service=myserv&job=1"
```
```json
{"status":{"code":200,"msg":"OK"},"head":{"method":"/train","job":1,"status":"running","time":74.0},"body":{"measure":{"iteration":445.0,"train_loss":0.7159726023674011,"mcll":2.1306082640485237,"acc":0.16127989657401424}}}
```

Returns information on a training job running asynchronously

### HTTP Request

`GET /train`

### Query Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
service | string | no | N/A | name of the service the training job is running on
job | int | no | N/A | job identifier
timeout | int | yes | 0 | timeout before the status is obtained
parameters.output.measure_hist | bool | yes | false | whether to return the full measure history until current point, useful for plotting

## Delete a training job

```shell
curl -X DELETE "http://localhost:8080/train?service=myserv&job=1"
```
```json
{"status":{"code":200,"msg":"OK"},"head":{"time":196.0,"status":"terminated","method":"/train","job":1}}
```

Kills a training job running asynchronously

### HTTP Request

`DELETE /train`

### Query Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
service | string | no | N/A | name of the service the training job is running on
job | int | no | N/A | job identifier


# Predict

Makes predictions from data out of an existing statistical model

## Prediction from service

> Prediction from image URL:

```shell
curl -X POST "http://localhost:8080/predict" -d "{\"service\":\"imageserv\",\"parameters\":{\"input\":{\"width\":224,\"height\":224},\"output\":{\"best\":3}},\"data\":[\"http://i.ytimg.com/vi/0vxOhd4qlnA/maxresdefault.jpg\"]}"
```
```json
{"status":{"code":200,"msg":"OK"},"head":{"method":"/predict","time":1591.0,"service":"imageserv"},"body":{"predictions":{"uri":"http://i.ytimg.com/vi/0vxOhd4qlnA/maxresdefault.jpg","loss":0.0,"classes":[{"prob":0.24278657138347627,"cat":"n03868863 oxygen mask"},{"prob":0.20703653991222382,"cat":"n03127747 crash helmet"},{"prob":0.07931024581193924,"cat":"n03379051 football helmet"}]}}}
```

> Prediction from CSV file:

```shell
curl -X POST "http://localhost:8080/predict" -d "{\"service\":\"covert\",\"parameters\":{\"input\":{\"id\":\"Id\",\"separator\":\",\",\"scale\":true}},\"data\":[\"models/covert/test10.csv\"]}"

```
```json
{"status":{"code":200,"msg":"OK"},"head":{"method":"/predict","time":16.0,"service":"covert"},"body":{"predictions":[{"uri":"15121","loss":0.0,"classes":{"prob":0.9999997615814209,"cat":"6"}},{"uri":"15122","loss":0.0,"classes":{"prob":0.9962882995605469,"cat":"5"}},{"uri":"15130","loss":0.0,"classes":{"prob":0.9999340772628784,"cat":"1"}},{"uri":"15123","loss":0.0,"classes":{"prob":1.0,"cat":"3"}},{"uri":"15124","loss":0.0,"classes":{"prob":1.0,"cat":"3"}},{"uri":"15128","loss":0.0,"classes":{"prob":1.0,"cat":"1"}},{"uri":"15125","loss":0.0,"classes":{"prob":0.9999998807907105,"cat":"3"}},{"uri":"15126","loss":0.0,"classes":{"prob":0.7535045146942139,"cat":"3"}},{"uri":"15129","loss":0.0,"classes":{"prob":0.9999986886978149,"cat":"1"}},{"uri":"15127","loss":0.0,"classes":{"prob":1.0,"cat":"1"}}]}}
```

Make predictions from data

### HTTP Request

`POST /predict`

### Query Parameters

#### General

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
service | string | no | N/A | name of the service to make predictions from
data | array of strings | no | N/A | array of data URI over which to make predictions

#### Input Connectors

- Image

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
width | int | yes | 227 | Resize images to width ("image" only)
height | int | yes | 227 | Resize images to height ("image" only)
bw | bool | yes | false | Treat images as black & white

- CSV

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
ignore | array of string | yes | empty | Array of column names to ignore
separator | string | yes | ',' | Column separator character
id | string | yes | empty | Column name of the training examples identifier field, if any
scale | bool | yes | false | Whether to scale all values into [0,1]

#### Output

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
template | string | yes | empty | Output template

TODO: see output template variables

#### Machine learning libraries

- Caffe

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
gpu | bool | yes | false | Whether to use GPU
gpuid | int | yes | 0 | GPU id to use

# Kittens
## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Isis",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/3"
  -H "Authorization: meowmeowmeow"
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Isis",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">If you're not using an administrator API key, note that some kittens will return 403 Forbidden if they are hidden for admins only.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the cat to retrieve
