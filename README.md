# SageMaker LLM Deployment Demo

这个项目展示了如何在Amazon SageMaker上部署和使用大型语言模型(LLMs)。项目使用Hugging Face LLM Inference Container来部署开源LLM模型，并提供了交互式的Gradio界面进行对话。

## 功能特点

- 使用Hugging Face Text Generation Inference (TGI)进行高性能文本生成
- 支持张量并行和动态批处理
- 提供量化选项以优化部署成本
- 包含交互式Gradio聊天界面
- 支持多种推理参数配置

## 环境要求

- AWS账号和相应的IAM权限
- Python 3.7+
- SageMaker Python SDK
- Gradio
- Boto3

## 支持的模型

目前项目中实现了以下模型的部署示例：

1. OpenAssistant/pythia-12b-sft-v8-7k-steps
2. eachadea/vicuna-7b-1.1

## 部署步骤

1. 环境设置
```python
import sagemaker
import boto3
from sagemaker.huggingface import get_huggingface_llm_image_uri

# 配置SageMaker会话
sess = sagemaker.Session()
role = sagemaker.get_execution_role()
```

2. 获取Hugging Face LLM容器
```python
llm_image = get_huggingface_llm_image_uri(
    "huggingface",
    version="0.6.0"
)
```

3. 创建和部署模型
```python
from sagemaker.huggingface import HuggingFaceModel

# 配置模型参数
llm_model = HuggingFaceModel(
    role=role,
    image_uri=llm_image,
    env={
        'HF_MODEL_ID': hf_model_id,
        'HF_MODEL_QUANTIZE': json.dumps(use_quantization),
        'SM_NUM_GPUS': json.dumps(number_of_gpu)
    }
)

# 部署模型
llm = llm_model.deploy(
    initial_instance_count=1,
    instance_type=instance_type,
    container_startup_health_check_timeout=health_check_timeout
)
```

## 使用方法

### Python SDK调用
```python
response = llm.predict({
    "inputs": "Your prompt here",
    "parameters": {
        "temperature": 0.7,
        "max_length": 200,
        "top_p": 0.7,
        "top_k": 50,
        "repetition_penalty": 1.03
    }
})
```

### Boto3调用
```python
import boto3
client = boto3.client('sagemaker-runtime')

response = client.invoke_endpoint(
    EndpointName='your-endpoint-name',
    ContentType='application/json',
    Accept='application/json',
    Body=json.dumps({
        "inputs": "Your prompt here",
        "parameters": {
            "temperature": 0.6,
            "max_length": 200
        }
    })
)
```

## 示例展示

### OpenAssistant/pythia-12B
![pythia-12B](https://github.com/VerRan/sagemaker-llm/blob/main/Screenshot%202023-06-01%20at%2018.14.53.png "pythia-12B")

### eachadea/vicuna-7b-1.1
![vicuna-7B](https://github.com/VerRan/sagemaker-llm/blob/main/Screenshot%202023-06-01%20at%2018.15.08.png "vicuna7B")

## 清理资源

在使用完毕后，可以通过以下命令删除部署的模型和端点：
```python
llm.delete_model()
llm.delete_endpoint()
```

## 注意事项

1. 部署时间可能需要10-15分钟
2. 建议使用g5.2xlarge或更高配置的实例类型
3. 可以通过开启量化(quantization)来优化部署成本
4. 确保AWS账号有足够的权限和配额

## 参考资料

- [Hugging Face Text Generation Inference](https://github.com/huggingface/text-generation-inference)
- [SageMaker Python SDK文档](https://sagemaker.readthedocs.io/)
- [SageMaker IAM角色配置](https://docs.aws.amazon.com/sagemaker/latest/dg/sagemaker-roles.html)
