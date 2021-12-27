# ML2
deep learning classification deployement on awsusing ECR Lambda REST APIGateway

**Ref:**
  * https://github.com/alexeygrigorev?tab=repositories
  * https://www.youtube.com/watch?v=Ndwju5Camds
  * https://github.com/alexeygrigorev/mlbookcamp-code/tree/master/chapter-08-serverless
  * https://pypi.org/project/keras-image-helper/
  * https://stackoverflow.com/questions/45594707/what-is-pips-no-cache-dir-good-for
  * https://github.com/alexeygrigorev/serverless-deep-learning

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


![image](https://user-images.githubusercontent.com/17021017/147505081-1973ffb2-c331-47c5-972a-b9310d45ccfb.png)




![image](https://user-images.githubusercontent.com/17021017/147503695-658b2524-6292-471f-9067-a489efbce4af.png)

![image](https://user-images.githubusercontent.com/17021017/147503737-61956493-4641-4dc0-996c-5231de22d5e4.png)

![image](https://user-images.githubusercontent.com/17021017/147503864-b8da7916-2a78-422b-a7ea-20f16979524e.png)

![image](https://user-images.githubusercontent.com/17021017/147503895-940c735a-cde7-46bd-ae1b-31332d21d457.png)

![image](https://user-images.githubusercontent.com/17021017/147503958-995cfb1b-35c9-4b05-ac22-c80c0b721e7b.png)

Loing to docker:

$ aws ecr get-login --no-include-email

![image](https://user-images.githubusercontent.com/17021017/147504109-32aae67a-5fb7-491f-afc7-65a93f8e171b.png)

Publish the image:

* REGION=eu-west-1
* ACCOUNT=XXXXXXXXXXXX
* REMOTE_NAME=${ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com/deep-classification-ecr:tf-lite-lambda 
* docker tag tf-lite-lambda ${REMOTE_NAME}
* docker push ${REMOTE_NAME}

Change REGION with your region and ACCOUNT with your ACCOUNT ID 

![image](https://user-images.githubusercontent.com/17021017/147504359-d5ca59c4-9028-45a6-a8c5-aa013225777c.png)













