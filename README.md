# Tensorflow-Object-Detection-API-on-AWS-Greengrass

This tutorial demonstrate how to use AWS greengrass with Tensorflow Object Detection API. Amazon provides a nice tutorial when it comes to using MXNet for image classification (Perform Machine Learning Inference), however the current tutorial focuses on steps needed to use Tensorflow Object Detection API on greengrass using pre-trained model with either own custom dataset or any open-source model like coco model.
If you are new to Amazon GreenGrass environment then I would suggest you to start with their first tutorial to get familiarize with the environment before going to machine learning on green-grass. Amazon has provided amazing tutorial to start from scratch here, this tutorial demonstrate how to setup green-grass on Raspberry-pi however similar steps can be applied if you want to do the setup on Ubuntu. I have used Ubuntu 18.04 on a laptop without gpu, as a edge device instead of Raspberry-pi.
Module 1 in the tutorial basically explains set-up process, in our case since we use ubuntu18 as edge we can directly jump to step 9 for adding user to group.

    	sudo adduser --system ggc_user
    	sudo addgroup --system ggc_group

After step 9 we can safely jump to step  12 and install AWS green-grass.

	git clone https://github.com/aws-samples/aws-greengrass-samples.git
	cd aws-greengrass-samples
	cd greengrass-dependency-checker-GGCv1.6.0
	sudo modprobe configs
	sudo ./check_ggc_dependencies | more

It is important to have a look at the warning mentioned by AWS after this step, which is as follows :
Because this tutorial only uses the AWS IoT Device SDK for Python, you can ignore warnings about the missing optional NodeJS 6.10 and Java 8 prerequisites that the check_ggc_dependencies script may produce.

After these steps, we can proceed to Module 2, all the steps mentioned in the tutorial can be followed as it is with only a minor modification in the step 8 in Configure AWS Greengrass on AWS IoT. Since we are using ubuntu18 as edge, we need to download AWS Greengrass core software for ubuntu 14-16 (Yes, I used the same for ubuntu 18 and it works just fine). After installation is complete we can finally start AWS Greengrass. Make sure that you change the tab to UNIX-like system when following the steps mentioned in the tutorial Start AWS Greengrass on the Core Device. If everything is fine, we will have AWS green-grass core running on our ubuntu 18 machine and now we can create and deploy our desired modules.
    I would suggest all of you to first do the module 3 Lambda Functions on AWS Greengrass as we are going to use lambda functions for machine learning inference, also this is good checkpoint to make sure that our installation is working file and we have basic pipeline ready. When I followed this tutorial, I got a small error saying there is no module greengrassHelloWorld. I fixed this error by making a modification in step 5, instead of naming the zip file as hello_world_python_lambda.zip, I named it as greengrassHelloWorld.zip and then I managed to run everything and see the messages received on test page of AWS console. I followed all other steps in the tutorial as it is and in the end everything  worked.
    So, after all this is done we have AWS green-grass core running and we also have a simple module deployed on it which sends us some messages. Now we can move forward towards our main topic of interest, running machine learning inference using tensorflow object detection API on AWS green-grass. As I mentioned before Amazon provides a tutorial for using MXNet for object classification and we will use that as our reference tutorial.
    We can completely ignore Step 1: Configure the Raspberry Pi, because we don’t use raspberry-pi and camera connected to it. Step 2 explains installation of the MXNet Framework so we can skip this step as well. Instead we need to install tensorflow and related packages needed for object detection API. Following steps can be done to complete this installation :

	sudo su
	pip install numpy pillow lxml jupyter matplotlib tensorflow pandas

Important : since I am using only cpu, tensorflow cpu version is installed. I think simple change for gpu version should work on gpu machine but I did not try that yet.

Step 3 is about creation of an MXNet Model Package, similarly we need to create model package for tensorflow model. You can use your own model trained with custom dataset or you can download a model from Tensorflow detection model zoo. I used ssd_mobilenet_v1_coco model trained on coco dataset from model zoo. Once we have the model as frozen_inference_graph.pb we need to create a zip package to  be able to upload it to AWS. Create a zip file with model and name it as tf_models.zip. Step 6 will explain how to upload this model to AWS.

Step 4 explains steps needed to Create and Publish a Lambda Function, this is basically our code which will perform inference on the model created in step 3. Since we did not do step 2 using mxnet we will not have greengrassObjectClassification.zip file ready as mentioned in this tutorial, however we will create our own module for object detection. Since tensorflow object detection API uses a lot of supporting files from their git repo and AWS lambda function allows code upto 25 MB space maximum we need to select only the files which are needed for object detection. All the supporting files along with the code to perform object detection is available on my Git repo. File greengrassObjectDetection.py performs object detection on images located in the local images folder and sends results to cloud on AWS. After you have downloaded all the files from my repo, create a zip file named greengrassObjectDetection.zip. After that follow all the steps mentioned in Step 4 in tutorial,  just make sure to replace greengrassObjectClassification name with greengrassObjectDetection wherever mentioned in the tutorial.

Perform step 5 as it is and add our lambda function created in step 4 to the group (greengrassObjectDetection). Since we will not use any camera or other resources but instead we will perform object detection on the images stored in the local folder we need to execute only few steps mentioned in Step 6: Add Resources to the Greengrass Group. To be specific we will skip steps 1 to 10 mentioned for creating videoCoreSharedMemory and videoCoreInterface. We will do the steps to add the inference model as a machine learning resource. This step includes uploading the tf_models.zip model package to Amazon S3.

I will repeat the steps below to avoid any confusion :

1. On the Machine Learning tab, choose Add machine learning resource.
2. On the Create a machine learning resource page, for Resource name, type tf_mobilenet_model.
3. For Model source, choose Locate or upload a model in S3.
4. Under Model from S3, choose Select, and then choose Create S3 bucket.
5. For Bucket name, type a name that contains the string greengrass (such as greengrass-datetime), and then choose Create.
	Note Don't use a period (".") in the bucket name.
6. Choose Upload a model, and then choose the tf_models.zip package that we created in Step 3: Create Model Package.
7. For Local path, type /greengrass-machine-learning/tf.
This is the destination for the local model in the Lambda runtime namespace. When you deploy the group, AWS Greengrass retrieves the source model package and then extracts the contents to the specified directory. The sample Lambda function for this tutorial is already configured to use this path (in the model_path variable).
8. Under Lambda function affiliations, choose Select.
9. Choose greengrassObjectDetection, choose Read-only access, and then choose Done.
10. At the bottom of the page, choose Save.

After all these steps we have all our modules ready in AWS cloud and we can perform Step 8 to Deploy the Greengrass Group as it is from the tutorial. Thats it finally we have our object detector running and this shoudl be sending the results to cloud. We can also do all the the steps as it is in section Test the Inference App and we will be able to see the detection results.

