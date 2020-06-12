# Inference of Mobilenetv2 Model on AWS SageMaker
This project is a deployed and pretrained Movilenetv2 model on CIFAR10, by huyvnphan on [https://github.com/huyvnphan/PyTorch_CIFAR10](https://github.com/huyvnphan/PyTorch_CIFAR10), 
and is hosted on AWS SageMaker and can be interacted via AWS API Gateway

![Endpoint](docs/sagemaker-endpoint-1.gif)

## To use the deployed model
Since the endpoints of the deployed models on AWS SageMaker are scoped to an individual account, and can not be made public, to use the deployed model, you can either:
1. Create an endpoint on your account. OR
2. Use HTTP request via AWS API Gateway and Lambda function implemented for a deployed endpoint.

This model recieves a image URL and gives prediction in a form of probabilities of the top three categories that this model deems it belongs to.

### To use the model endpoint on your own account
1. Open the Amazon SageMaker console at [https://console.aws.amazon.com/sagemaker/](https://console.aws.amazon.com/sagemaker/).

2. Create a notebook instance.<br/>
a. For **Instance Type** choose `ml.t2.medium`<br/>
b. For **IAM role**, choose **Create a new role**, then choose **Create role**.

3. Upload `model/code/inference.py` in this repository to the notebook instance, create a `Deploy.ipynb` in the same directory.

4. In `Deploy.ipynb`, create an endpoint by running:
	```python
	from sagemaker import get_execution_role
	from sagemaker.pytorch import PyTorchModel
	
	role = get_execution_role()

	pytorch_model = PyTorchModel(model_data='s3://sagemaker-fova-bucket/model.tar.gz', role=role,
	                             entry_point='inference.py', framework_version='1.4.0')

	predictor = pytorch_model.deploy(instance_type='ml.t2.medium', initial_instance_count=1)
	```
	*Note:*<br/>
	a. This process will take about 15 mins.<br/>
	b. Since this is a sample project, I set the read permission of the model file on S3 to public, so you can use it directly.
	
5. Get real time predictions by calling an inference endpoint and passing a request payload in JSON format.<br/>
	***Please remember to change the EndpointName field.***
	```python
	import boto3
	import json

	runtime = boto3.client('runtime.sagemaker')
	
	# pass a sample CIFAR10 image URL in json formatt to  
	payload = json.dumps({'url':'https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcRVTpWyHJ2HzwwqK8HELZqRzmLQykp4aVbMsTlkB2sPIGIC7dCU&usqp=CAU'})
	
	response = runtime.invoke_endpoint(EndpointName='<CHANGE TO YOUR ENDPOINT NAME>',
	                                   ContentType='application/json',
	                                   Body=payload)

	result = json.loads(response['Body'].read().decode())
	print(result)
	```

### To use the model directly via HTTP request (No prior steps needed)

![Post](docs/post.png)
In Postman, POST an HTTP request:
`https://o7ha4cifzj.execute-api.us-west-1.amazonaws.com/test/predict`

In the request body,  input your data point
`{"url":"<your image url>"}`
Sample image url:
`https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcRVTpWyHJ2HzwwqK8HELZqRzmLQykp4aVbMsTlkB2sPIGIC7dCU&usqp=CAU`

The result will be returned in the response.
