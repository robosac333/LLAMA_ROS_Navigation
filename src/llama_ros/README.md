# llama_ros

This repository provides a set of ROS 2 packages to integrate [llama.cpp](https://github.com/ggerganov/llama.cpp) into ROS 2. Using the llama_ros packages, you can easily incorporate the powerful optimization capabilities of [llama.cpp](https://github.com/ggerganov/llama.cpp) into your ROS 2 projects by running [GGUF](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md)-based [LLMs](https://huggingface.co/models?sort=trending&search=gguf+7b) and [VLMs](https://huggingface.co/models?sort=trending&search=gguf+llava).

## Table of Contents

1. [Related Projects](#related-projects)
2. [Installation](#installation)
3. [Usage](#usage)
   - [llama_cli](#llama_cli)
   - [Launch Files](#launch-files)
   - [LoRA Adapters](#lora-adapters)
   - [ROS 2 Clients](#ros-2-clients)
   - [LangChain](#langchain)
4. [Demos](#demos)

## Related Projects

- [chatbot_ros](https://github.com/mgonzs13/chatbot_ros) &rarr; This chatbot, integrated into ROS 2, uses [whisper_ros](https://github.com/mgonzs13/whisper_ros/tree/main), to listen to people speech; and llama_ros, to generate responses. The chatbot is controlled by a state machine created with [YASMIN](https://github.com/uleroboticsgroup/yasmin).
- [explainable_ros](https://github.com/Dsobh/explainable_ROS) &rarr; A ROS 2 tool to explain the behavior of a robot. Using the integration of LangChain, logs are stored in a vector database. Then, RAG is applied to retrieve relevant logs for user questions answered with llama_ros.

## Installation

To run llama_ros with CUDA, first, you must install the [CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit).

```shell
$ cd ~/ros2_ws/src
$ git clone https://github.com/mgonzs13/llama_ros.git
$ pip3 install -r llama_ros/requirements.txt
$ cd ~/ros2_ws
$ rosdep install --from-paths src --ignore-src -r -y
$ colcon build --cmake-args -DGGML_CUDA=ON # add this for CUDA
```

## Usage

### llama_cli

Commands are included in llama_ros to speed up the test of GGUF-based LLMs within the ROS 2 ecosystem. This way, the following commands are integrating into the ROS 2 commands:

#### launch

Using this command launch a LLM from a YAML file. The configuration of the YAML is used to launch the LLM in the same way as using a regular launch file. Here is an example of how to use it:

```shell
$ ros2 llama launch ~/ros2_ws/src/llama_ros/llama_bringup/params/StableLM-Zephyr.yaml
```

#### prompt

Using this command send a prompt to a launched LLM. The command uses a string, which is the prompt and has the following arguments:

- (`-r`, `--reset`): Whether to reset the LLM before prompting
- (`-t`, `--temp`): The temperature value
- (`--image-url`): Image url to sent to a VLM

Here is an example of how to use it:

```shell
$ ros2 llama prompt "Do you know ROS 2?" -t 0.0
```

### Launch Files

First of all, you need to create a launch file to use llama_ros or llava_ros. This launch file will contain the main parameters to download the model from HuggingFace and configure it. Take a look at the following examples and the [predefined launch files](llama_bringup/launch).

#### llama_ros (Python Launch)

<details>
<summary>Click to expand</summary>

```python
from launch import LaunchDescription
from llama_bringup.utils import create_llama_launch


def generate_launch_description():

    return LaunchDescription([
        create_llama_launch(
            n_ctx=2048, # context of the LLM in tokens
            n_batch=8, # batch size in tokens
            n_gpu_layers=0, # layers to load in GPU
            n_threads=1, # threads
            n_predict=2048, # max tokens, -1 == inf

            model_repo="TheBloke/Marcoroni-7B-v3-GGUF", # Hugging Face repo
            model_filename="marcoroni-7b-v3.Q4_K_M.gguf", # model file in repo

            system_prompt_type="alpaca" # system prompt type
        )
    ])
```

```shell
$ ros2 launch llama_bringup marcoroni.launch.py
```

</details>

#### llama_ros (YAML Config)

<details>
<summary>Click to expand</summary>

```yaml
n_ctx: 2048 # context of the LLM in tokens
n_batch: 8 # batch size in tokens
n_gpu_layers: 0 # layers to load in GPU
n_threads: 1 # threads
n_predict: 2048 # max tokens, -1 == inf

model_repo: "cstr/Spaetzle-v60-7b-GGUF" # Hugging Face repo
model_filename: "Spaetzle-v60-7b-q4-k-m.gguf" # model file in repo

system_prompt_type: "Alpaca" # system prompt type
```

```python
import os
from launch import LaunchDescription
from llama_bringup.utils import create_llama_launch_from_yaml
from ament_index_python.packages import get_package_share_directory


def generate_launch_description():
    return LaunchDescription([
        create_llama_launch_from_yaml(os.path.join(
            get_package_share_directory("llama_bringup"), "params", "Spaetzle.yaml"))
    ])
```

```shell
$ ros2 launch llama_bringup spaetzle.launch.py
```

</details>

#### llava_ros (Python Launch)

<details>
<summary>Click to expand</summary>

```python
from launch import LaunchDescription
from llama_bringup.utils import create_llama_launch

def generate_launch_description():

    return LaunchDescription([
        create_llama_launch(
            use_llava=True, # enable llava

            n_ctx=8192, # context of the LLM in tokens, use a huge context size to load images
            n_batch=512, # batch size in tokens
            n_gpu_layers=33, # layers to load in GPU
            n_threads=1, # threads
            n_predict=8192, # max tokens, -1 == inf

            model_repo="cjpais/llava-1.6-mistral-7b-gguf", # Hugging Face repo
            model_filename="llava-v1.6-mistral-7b.Q4_K_M.gguf", # model file in repo

            mmproj_repo="cjpais/llava-1.6-mistral-7b-gguf", # Hugging Face repo
            mmproj_filename="mmproj-model-f16.gguf", # mmproj file in repo

            system_prompt_type="mistral" # system prompt type
        )
    ])
```

```shell
$ ros2 launch llama_bringup llava.launch.py
```

</details>

#### llava_ros (YAML Config)

<details>
<summary>Click to expand</summary>

```yaml
use_llava: True # enable llava

n_ctx: 8192 # context of the LLM in tokens use a huge context size to load images
n_batch: 512 # batch size in tokens
n_gpu_layers: 33 # layers to load in GPU
n_threads: 1 # threads
n_predict: 8192 # max tokens -1 : :  inf

model_repo: "cjpais/llava-1.6-mistral-7b-gguf" # Hugging Face repo
model_filename: "llava-v1.6-mistral-7b.Q4_K_M.gguf" # model file in repo

mmproj_repo: "cjpais/llava-1.6-mistral-7b-gguf" # Hugging Face repo
mmproj_filename: "mmproj-model-f16.gguf" # mmproj file in repo

system_prompt_type: "mistral" # system prompt type
```

```python
def generate_launch_description():
    return LaunchDescription([
        create_llama_launch_from_yaml(os.path.join(
            get_package_share_directory("llama_bringup"),
            "params", "llava-1.6-mistral-7b-gguf.yaml"))
    ])
```

```shell
$ ros2 launch llama_bringup llava.launch.py
```

</details>

### LoRA Adapters

You can use LoRA adapters when launching LLMs. Using llama.cpp features, you can load multiple adapters choosing the scale to apply for each adapter. Here you have an example of using LoRA adapters with Phi-3. You can lis the
LoRAs using the `/llama/list_loras` service and modify their scales values by using the `/llama/update_loras` service. A scale value of 0.0 means not using that LoRA.

<details>
<summary>Click to expand</summary>

```yaml
n_ctx: 2048
n_batch: 8
n_gpu_layers: 0
n_threads: 1
n_predict: 2048

model_repo: "bartowski/Phi-3.5-mini-instruct-GGUF"
model_filename: "Phi-3.5-mini-instruct-Q4_K_M.gguf"

lora_adapters:
  - repo: "zhhan/adapter-Phi-3-mini-4k-instruct_code_writing"
    filename: "Phi-3-mini-4k-instruct-adaptor-f16-code_writer.gguf"
    scale: 0.5
  - repo: "zhhan/adapter-Phi-3-mini-4k-instruct_summarization"
    filename: "Phi-3-mini-4k-instruct-adaptor-f16-summarization.gguf"
    scale: 0.5

system_prompt_type: "Phi-3"
```

</details>

### ROS 2 Clients

Both llama_ros and llava_ros provide ROS 2 interfaces to access the main functionalities of the models. Here you have some examples of how to use them inside ROS 2 nodes. Moreover, take a look to the [llama_demo_node.py](llama_demos/llama_demos/llama_demo_node.py) and [llava_demo_node.py](llama_demos/llama_demos/llava_demo_node.py) demos.

#### Tokenize

<details>
<summary>Click to expand</summary>

```python
from rclpy.node import Node
from llama_msgs.srv import Tokenize


class ExampleNode(Node):
    def __init__(self) -> None:
        super().__init__("example_node")

        # create the client
        self.srv_client = self.create_client(Tokenize, "/llama/tokenize")

        # create the request
        req = Tokenize.Request()
        req.prompt = "Example text"

        # call the tokenize service
        self.srv_client.wait_for_service()
        res = self.srv_client.call(req)
        tokens = res.tokens
```

</details>

#### Embeddings

<details>
<summary>Click to expand</summary>

_Remember to launch llama_ros with embedding set to true to be able of generating embeddings with your LLM._

```python
from rclpy.node import Node
from llama_msgs.srv import Embeddings


class ExampleNode(Node):
    def __init__(self) -> None:
        super().__init__("example_node")

        # create the client
        self.srv_client = self.create_client(Embeddings, "/llama/generate_embeddings")

        # create the request
        req = Embeddings.Request()
        req.prompt = "Example text"
        req.normalize = True

        # call the embedding service
        self.srv_client.wait_for_service()
        res = self.srv_client.call(req)
        embeddings = res.embeddings
```

</details>

#### Generate Response

<details>
<summary>Click to expand</summary>

```python
import rclpy
from rclpy.node import Node
from rclpy.action import ActionClient
from llama_msgs.action import GenerateResponse


class ExampleNode(Node):
    def __init__(self) -> None:
        super().__init__("example_node")

        # create the client
        self.action_client = ActionClient(
            self, GenerateResponse, "/llama/generate_response")

        # create the goal and set the sampling config
        goal = GenerateResponse.Goal()
        goal.prompt = self.prompt
        goal.sampling_config.temp = 0.2

        # wait for the server and send the goal
        self.action_client.wait_for_server()
        send_goal_future = self.action_client.send_goal_async(
            goal)

        # wait for the server
        rclpy.spin_until_future_complete(self, send_goal_future)
        get_result_future = send_goal_future.result().get_result_async()

        # wait again and take the result
        rclpy.spin_until_future_complete(self, get_result_future)
        result: GenerateResponse.Result = get_result_future.result().result
```

</details>

#### Generate Response (llava)

<details>
<summary>Click to expand</summary>

```python
import cv2
from cv_bridge import CvBridge

import rclpy
from rclpy.node import Node
from rclpy.action import ActionClient
from llama_msgs.action import GenerateResponse


class ExampleNode(Node):
    def __init__(self) -> None:
        super().__init__("example_node")

        # create a cv bridge for the image
        self.cv_bridge = CvBridge()

        # create the client
        self.action_client = ActionClient(
            self, GenerateResponse, "/llama/generate_response")

        # create the goal and set the sampling config
        goal = GenerateResponse.Goal()
        goal.prompt = self.prompt
        goal.sampling_config.temp = 0.2

        # add your image to the goal
        image = cv2.imread("/path/to/your/image", cv2.IMREAD_COLOR)
        goal.image = self.cv_bridge.cv2_to_imgmsg(image)

        # wait for the server and send the goal
        self.action_client.wait_for_server()
        send_goal_future = self.action_client.send_goal_async(
            goal)

        # wait for the server
        rclpy.spin_until_future_complete(self, send_goal_future)
        get_result_future = send_goal_future.result().get_result_async()

        # wait again and take the result
        rclpy.spin_until_future_complete(self, get_result_future)
        result: GenerateResponse.Result = get_result_future.result().result
```

</details>

### LangChain

There is a [llama_ros integration for LangChain](llama_ros/llama_ros/langchain/). Thus, prompt engineering techniques could be applied. Here you have an example to use it.

#### llama_ros (Chain)

<details>
<summary>Click to expand</summary>

```python
import rclpy
from llama_ros.langchain import LlamaROS
from langchain.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser


rclpy.init()

# create the llama_ros llm for langchain
llm = LlamaROS()

# create a prompt template
prompt_template = "tell me a joke about {topic}"
prompt = PromptTemplate(
    input_variables=["topic"],
    template=prompt_template
)

# create a chain with the llm and the prompt template
chain = prompt | llm | StrOutputParser()

# run the chain
text = chain.invoke({"topic": "bears"})
print(text)

rclpy.shutdown()
```

</details>

#### llama_ros (Stream)

<details>
<summary>Click to expand</summary>

```python
import rclpy
from llama_ros.langchain import LlamaROS
from langchain.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser


rclpy.init()

# create the llama_ros llm for langchain
llm = LlamaROS()

# create a prompt template
prompt_template = "tell me a joke about {topic}"
prompt = PromptTemplate(
    input_variables=["topic"],
    template=prompt_template
)

# create a chain with the llm and the prompt template
chain = prompt | llm | StrOutputParser()

# run the chain
for c in chain.stream({"topic": "bears"}):
    print(c, flush=True, end="")

rclpy.shutdown()
```

</details>

### llava_ros

<details>
<summary>Click to expand</summary>

```python
import rclpy
from llama_ros.langchain import LlamaROS

rclpy.init()

# create the llama_ros llm for langchain
llm = LlamaROS()

# bind the url_image
llm = llm.bind(image_url=image_url).stream("Describe the image")
image_url = "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"

# run the llm
for c in llm:
    print(c, flush=True, end="")

rclpy.shutdown()

```

</details>

#### llama_ros_embeddings (RAG)

<details>
<summary>Click to expand</summary>

```python
import rclpy
from langchain_community.vectorstores import Chroma
from llama_ros.langchain import LlamaROSEmbeddings


rclpy.init()

# create the llama_ros embeddings for lanchain
embeddings = LlamaROSEmbeddings()

# create a vector database and assign it
db = Chroma(embedding_function=embeddings)

# create the retriever
retriever = db.as_retriever(search_kwargs={"k": 5})

# add your texts
db.add_texts(texts=["your_texts"])

# retrieve documents
docuemnts = retriever.get_relevant_documents("your_query")
print(docuemnts)

rclpy.shutdown()
```

</details>

## Demos

### llama_ros

```shell
$ ros2 launch llama_bringup spaetzle.launch.py
```

```shell
$ ros2 run llama_demos llama_demo_node --ros-args -p prompt:="your prompt"
```

<!-- https://user-images.githubusercontent.com/25979134/229344687-9dda3446-9f1f-40ab-9723-9929597a042c.mp4 -->

https://github.com/mgonzs13/llama_ros/assets/25979134/9311761b-d900-4e58-b9f8-11c8efefdac4

### llava_ros

```shell
$ ros2 launch llama_bringup minicpm-2.6.launch.py
```

```shell
$ ros2 run llama_demos llava_demo_node --ros-args -p prompt:="your prompt" -p image_url:="url of the image" -p use_image:="whether to send the image"
```

https://github.com/mgonzs13/llama_ros/assets/25979134/4a9ef92f-9099-41b4-8350-765336e3503c

### chatllama

```shell
$ ros2 launch llama_bringup minicpm-2.6.launch.py
```

```shell
$ ros2 run llama_demos chatllama_demo_node
```

[ChatLlamaROS demo](https://github-production-user-asset-6210df.s3.amazonaws.com/55236157/363094669-c6de124a-4e91-4479-99b6-685fecb0ac20.webm?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240830%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240830T081232Z&X-Amz-Expires=300&X-Amz-Signature=f937758f4bcbaec7683e46ddb057fb642dc86a33cc8c736fca3b5ce2bf06ddac&X-Amz-SignedHeaders=host&actor_id=55236157&key_id=0&repo_id=622137360)
