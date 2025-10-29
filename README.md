# Dio
Desafio: Orquestração de Workflows com AWS Step Functions (DIO)

Este projeto documenta a implementação de um fluxo de trabalho (workflow) serverless na AWS, utilizando o Step Functions para orquestrar a execução sequencial de três funções AWS Lambda.

Arquitetura e Fluxo
O workflow simula um processo de três etapas, garantindo que cada tarefa seja executada na ordem correta. A orquestração é gerenciada inteiramente pelo AWS Step Functions.
Fluxo de Execução:** `InicioDoProcesso` ➡️ `ExecutaTarefa` ➡️ `FinalizaComSucesso`

2. Serviços AWS Utilizados

AWS Step Functions:** Serviço principal para definir e gerenciar a sequência de estados (o workflow).
AWS Lambda:** Executa o código em cada passo do workflow. Utilizamos três funções simples (passos).
AWS IAM:** Gerencia as permissões, garantindo que o Step Functions possa invocar as funções Lambda.

Definição do Workflow ASL
{
  "Comment": "A description of my state machine",
  "StartAt": "Passo_1",
  "States": {
    "Passo_1": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Output": "{% $states.result.Payload %}",
      "Arguments": {
        "FunctionName": "arn:aws:lambda:us-east-2:787038811957:function:InicioDoProcesso:$LATEST",
        "Payload": "{% $states.input %}"
      },
      "Retry": [
        {
          "ErrorEquals": [
            "Lambda.ServiceException",
            "Lambda.AWSLambdaException",
            "Lambda.SdkClientException",
            "Lambda.TooManyRequestsException"
          ],
          "IntervalSeconds": 1,
          "MaxAttempts": 3,
          "BackoffRate": 2,
          "JitterStrategy": "FULL"
        }
      ],
      "Next": "Passo_2"
    },
    "Passo_2": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Output": "{% $states.result.Payload %}",
      "Arguments": {
        "FunctionName": "arn:aws:lambda:us-east-2:787038811957:function:ExecutaTarefa:$LATEST",
        "Payload": "{% $states.input %}"
      },
      "Retry": [
        {
          "ErrorEquals": [
            "Lambda.ServiceException",
            "Lambda.AWSLambdaException",
            "Lambda.SdkClientException",
            "Lambda.TooManyRequestsException"
          ],
          "IntervalSeconds": 1,
          "MaxAttempts": 3,
          "BackoffRate": 2,
          "JitterStrategy": "FULL"
        }
      ],
      "Next": "Passo_3"
    },
    "Passo_3": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Output": "{% $states.result.Payload %}",
      "Arguments": {
        "FunctionName": "arn:aws:lambda:us-east-2:787038811957:function:FinalizaComSucesso:$LATEST",
        "Payload": "{% $states.input %}"
      },
      "Retry": [
        {
          "ErrorEquals": [
            "Lambda.ServiceException",
            "Lambda.AWSLambdaException",
            "Lambda.SdkClientException",
            "Lambda.TooManyRequestsException"
          ],
          "IntervalSeconds": 1,
          "MaxAttempts": 3,
          "BackoffRate": 2,
          "JitterStrategy": "FULL"
        }
      ],
      "End": true
    }
  },
  "QueryLanguage": "JSONata"
}
