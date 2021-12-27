# ML2
My practice of deep learning classification deployement on aws using Lambda, ECR Docker, REST APIGateway with the help of Cloud9 IDE.

**Ref:**
  * https://github.com/alexeygrigorev?tab=repositories
  * https://www.youtube.com/watch?v=Ndwju5Camds
  * https://github.com/alexeygrigorev/mlbookcamp-code/tree/master/chapter-08-serverless
  * https://pypi.org/project/keras-image-helper/
  * https://stackoverflow.com/questions/45594707/what-is-pips-no-cache-dir-good-for
  * https://github.com/alexeygrigorev/serverless-deep-learning
  * https://github.com/alexeygrigorev/aws-lambda-docker/blob/main/guide.md

## **Steps**

# 1: 
Create aws account, login to account.

# 2: Create then open Cloud9  IDE

AWS Cloud9 is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. It includes a code editor, debugger, and terminal.

![image](https://user-images.githubusercontent.com/17021017/147494006-da1b51c4-eaa4-4ba6-b429-a21c61979082.png)

# 3: clone the code from https://github.com/alexeygrigorev/mlbookcamp-code.git to your Cloud9 IDE.

$ git clone https://github.com/alexeygrigorev/mlbookcamp-code.git

![image](https://user-images.githubusercontent.com/17021017/147496701-91360347-c680-456f-b909-ebe462c87667.png)

your project structure lokks like below:

![image](https://user-images.githubusercontent.com/17021017/147496789-37dc73c3-67e5-4ce6-a9a7-d49c897b4db8.png)

but our code is in the chapter-08-serverless folder so go to that folder

![image](https://user-images.githubusercontent.com/17021017/147496921-71f53ae0-9f3a-4a55-9447-a63c01e5ce1d.png)

# 4:
Get the model file:(download the model to your location using below command)

$wget https://github.com/alexeygrigorev/mlbookcamp-code/releases/download/chapter7-model/xception_v4_large_08_0.894.h5

![image](https://user-images.githubusercontent.com/17021017/147498115-0a9f9396-36e0-4d63-9cbf-00180f1407b2.png)

# 5: 
We can deploy the above model but its not a cost effective because to predict the model ,we need to load the image (load_img) and preprocess the image (preprocess_input) using tensorflow libraries.tensorflow takes more space so we are going to use tensorflow lite.

So convert above model as lensorfloe lite model using convert.py file.

Run using below command.

$python3.7 convert.py 

**Note**: 

1. In Cloud9 IDE,above command wont work so first install tensorflow.tensorflow also wont install because iam using t2.micro install(low space).If you are also using the same instance, run the  convert.py file in your local machine(or in kaggle) then drag and drop the resultant file clothing-model-v4.tflite to cloud9.
2. while running convert.py file ,keep xception_v4_large_08_0.894.h5 file in the same location then output will be clothing-model-v4.tflite.

![image](https://user-images.githubusercontent.com/17021017/147499640-48f22bd5-3be9-46e6-a53f-55f776d0ec52.png)

# 6:
Build the docker image:

### Docker file explanation

1. FROM public.ecr.aws/lambda/python:3.7

The **FROM** instruction initializes a new build stage and sets the Base Image for subsequent instructions. As such, a valid Dockerfile must start with a FROM instruction. The image can be any valid image – it is especially easy to start by pulling an image from the Public Repositories

2. RUN pip3 install --upgrade pip

The **RUN** instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile .

3. RUN pip3 install keras_image_helper --no-cache-dir

**keras_image_helper** is a lightweight library for pre-processing images for pre-trained keras models.

I think there is a good reason to use **--no-cache-dir** when you are building Docker images. The cache is usually useless in a Docker image, and you can definitely shrink the image size by disabling the cache.

4. RUN pip3 install https://raw.githubusercontent.com/alexeygrigorev/serverless-deep-learning/master/tflite/tflite_runtime-2.2.0-cp37-cp37m-linux_x86_64.whl --no-cache-dir

A **WHL** file is a package saved in the Wheel format, which is the standard built-package format used for Python distributions.

Install the ftlite from WHL file.

5. COPY clothing-model-v4.tflite clothing-model-v4.tflite

The **COPY** instruction lets us copy a file (or files) from the host system into the image.

6. COPY lambda_function.py lambda_function.py

7. CMD [ "lambda_function.lambda_handler" ]

There can only be one **CMD** instruction in a Dockerfile . If you list more than one CMD then only the last CMD will take effect. The main purpose of a CMD is to provide defaults for an executing container.

### Now build the image

Run the below commad 

$docker build -t tf-lite-lambda .  

Docker image created with the **tf-lite-lambda** name

![image](https://user-images.githubusercontent.com/17021017/147503252-a51c6371-b6f0-46ea-b6cc-59638b0c7f45.png)

# 7:Publishing

Create an ECR repo: (My repo name : **lambda-images** )

$aws ecr create-repository --repository-name lambda-images

![image](https://user-images.githubusercontent.com/17021017/147504857-12b874f3-cdd5-4f73-b4a5-b70fc70e826f.png)

You can create/view from console

![image](https://user-images.githubusercontent.com/17021017/147507039-73ba2c5b-4495-4512-b5e0-fda1c11fdc35.png)

Now login to docker then publish the image to ECR(Elastic Container Registry).

$ $(aws ecr get-login --no-include-email)

**Note: dont forget dollar symbol($)**

![image](https://user-images.githubusercontent.com/17021017/147506878-88c322c3-5cc8-4c5a-a428-f45f14cbc73a.png)

Publish the image:

* REGION=eu-west-1
* ACCOUNT=XXXXXXXXXXXX
* REMOTE_NAME=${ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com/deep-classification-ecr:tf-lite-lambda 
* docker tag tf-lite-lambda ${REMOTE_NAME}
* docker push ${REMOTE_NAME}

Change REGION with your region and ACCOUNT with your ACCOUNT ID 

![image](https://user-images.githubusercontent.com/17021017/147504359-d5ca59c4-9028-45a6-a8c5-aa013225777c.png)


![image](https://user-images.githubusercontent.com/17021017/147507413-92caa47d-d627-4bf2-bf95-8f03bd879df4.png)

Now you can the sameimage in the ECR console also.

![image](https://user-images.githubusercontent.com/17021017/147507461-2a286677-24ec-4bc0-8bbc-fe743945e331.png)

# 8: Create a lambda function

![image](https://user-images.githubusercontent.com/17021017/147508048-fe7f79f1-c9eb-4b14-8381-b19595f44c42.png)

![image](https://user-images.githubusercontent.com/17021017/147508089-ef5f03f3-1270-47ae-8359-eeddcf19a403.png)

![image](https://user-images.githubusercontent.com/17021017/147508110-455113d4-be46-4053-911e-67d200baa529.png)

![image](https://user-images.githubusercontent.com/17021017/147508225-82934350-d65d-4219-8577-3cc54e85549b.png)

![image](https://user-images.githubusercontent.com/17021017/147508260-05e2298a-16b2-4e5d-a99f-661c072d00cb.png)

![image](https://user-images.githubusercontent.com/17021017/147508294-756f4cc8-c371-4978-9253-81f15a513c2c.png)

![image](https://user-images.githubusercontent.com/17021017/147508368-6a5adfc7-a39e-407c-9f64-2716622ceff7.png)

![image](https://user-images.githubusercontent.com/17021017/147508405-e312fce6-78df-4346-94d5-d23dea32652d.png)

![image](https://user-images.githubusercontent.com/17021017/147508440-ea8f6058-7c5c-45f5-a7ce-5df931942350.png)

### Test the lambda function by passing image URL

![image](https://user-images.githubusercontent.com/17021017/147508521-a1cbe787-3a01-41c7-bf22-8d1851b3efb2.png)

### Results:

![image](https://user-images.githubusercontent.com/17021017/147508577-076a9764-9bbe-49c7-8785-34db3dbc8a63.png)


# 9: lambda_function.py explanation

lambda function execute the lambda_function.py file

![image](https://user-images.githubusercontent.com/17021017/147509009-f14d866b-c235-48bf-bbe8-82294db31e3e.png)

lambda starts the execution from **lambda_handler** function.It will take the url then preprocess,predict and decode the results then return the results.

![image](https://user-images.githubusercontent.com/17021017/147509254-995502ee-301c-4a2c-ae27-66585226946e.png)

# 10: Invoke the Lambda function from the API Gateway

![image](https://user-images.githubusercontent.com/17021017/147509468-67ddc178-c65c-4154-af88-42ad84b13d1a.png)

![image](https://user-images.githubusercontent.com/17021017/147509503-c0a4d46a-64e1-40f1-bcb9-c5d03d33890e.png)

![image](https://user-images.githubusercontent.com/17021017/147509554-7ac2fc08-0196-4d28-bafe-edc60a50691e.png)

![image](https://user-images.githubusercontent.com/17021017/147509590-305f28fa-d58c-4103-abb9-6eab35d8d5c3.png)

![image](https://user-images.githubusercontent.com/17021017/147509660-144c9eb7-9a08-48ae-8b53-5b9a94af4bbf.png)

![image](https://user-images.githubusercontent.com/17021017/147509684-191c994d-7b23-4516-9cd8-c595031c27cc.png)

![image](https://user-images.githubusercontent.com/17021017/147509711-127ea656-fd93-42da-9a44-fc5c41ad283d.png)

![image](https://user-images.githubusercontent.com/17021017/147509745-14bcbb62-4ae0-46d2-80c7-70b2f00153b9.png)

![image](https://user-images.githubusercontent.com/17021017/147509804-c18bfe3c-478b-41c7-a1ba-3e3c5019a39b.png)

![image](https://user-images.githubusercontent.com/17021017/147509858-fd1f423e-65cf-4ef0-a5a9-473d4298d154.png)

![image](https://user-images.githubusercontent.com/17021017/147509874-f8bf27f0-35f7-4cc0-8594-b62f0ec9044e.png)

![image](https://user-images.githubusercontent.com/17021017/147509944-c782f778-a220-46ab-9b88-033ec6c33193.png)

![image](https://user-images.githubusercontent.com/17021017/147509960-d7ec77ad-2296-4bea-add8-660e2af05b4e.png)

![image](https://user-images.githubusercontent.com/17021017/147509978-68030a1d-d286-4ea6-b382-4f524917f137.png)

Below one is the public url

https://ma25u2sof5.execute-api.ap-south-1.amazonaws.com/beta/prediction

# 11: Test with RESTer or postman

![image](https://user-images.githubusercontent.com/17021017/147510158-4bfbb333-4e92-4773-9e8d-66675e3d2d5b.png)












