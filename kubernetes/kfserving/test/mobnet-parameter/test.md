# Parameter Optimizing Test

1. Pre-requisites
    - alibi == 0.3.2
    - tensorflow == 1.15.3
    
2. Inference
    - 결과
        - batch_size: 20
            - 138576.46ms (compared to 148885.28ms without stop_on_firsts)
        - batch_size: 50
            - (transformer) http1connection:289] Malformed HTTP message from 127.0.0.1: Content-Length too long
            - (explainer) ('Connection aborted.', ConnectionResetError(104, 'Connection reset by peer'))
            - 502 bad gateway
    ```
    export INFERENCE_NS="default"
    export MODEL_NAME="mobnet-stop"
    export INPUT_PATH="@./input_data.json"
    
    export CLUSTER_IP=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    export SERVICE_HOSTNAME=$(kubectl -n $INFERENCE_NS get inferenceservice $MODEL_NAME -o jsonpath='{.status.url}' | cut -d "/" -f 3)
    echo "$CLUSTER_IP, $SERVICE_HOSTNAME"
    
    curl -v -H "Host: ${SERVICE_HOSTNAME}" http://$CLUSTER_IP/v1/models/$MODEL_NAME:predict -d $INPUT_PATH > ./output_predict.json
    curl -v -H "Host: ${SERVICE_HOSTNAME}" http://$CLUSTER_IP/v1/models/$MODEL_NAME:explain -d $INPUT_PATH > ./output_explain.json
    ```
   
3. log
    ```
    # transformer
    kubectl -n $INFERENCE_NS logs \
          "$(kubectl -n $INFERENCE_NS get pods -l model=$MODEL_NAME,component=transformer \
          -o jsonpath='{.items[0].metadata.name}')" -c kfserving-container > transformer.log
    # predictor 
    kubectl -n $INFERENCE_NS logs \
          "$(kubectl -n $INFERENCE_NS get pods -l model=$MODEL_NAME,component=predictor \
          -o jsonpath='{.items[0].metadata.name}')" -c kfserving-container > predictor.log
    # explainer
    kubectl -n $INFERENCE_NS logs \
          "$(kubectl -n $INFERENCE_NS get pods -l model=$MODEL_NAME,component=explainer \
          -o jsonpath='{.items[0].metadata.name}')" -c kfserving-container > explainer.log
    ```