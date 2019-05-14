Using Apache Bench:

For setup:
from clipper_admin import ClipperConnection, DockerContainerManager
clipper_conn = ClipperConnection(DockerContainerManager())
clipper_conn.start_clipper()
from sklearn.externals import joblib
model = joblib.load('/Users/shubhamadep/Documents/predicting_Tv_show/save_model_sklearn.pkl')
from clipper_admin.deployers import python as python_deployer
python_deployer.deploy_python_closure(clipper_conn, name="sum-model", version=1, input_type="int", pkgs_to_install = ["sklearn"], func=model.predict)
clipper_conn.register_application(name="pipeline-sklearn", input_type="int", default_output="-1.0", slo_micros=100000)
clipper_conn.link_model_to_app(app_name="pipeline-sklearn", model_name="test-model")

### Now the model is deployed on local server. 
Cache was disabled in the code. Hence same request was sent multiple times, and concurrently.

For final test used:
ab -p ./json_input.txt -T application/json -c 20 -n 100000 127.0.0.1:1337/pipeline-sklearn/predict

For inital model testing without Apache Bench:
time for i in `seq 1 20`; do curl -X POST --header "Content-Type:application/json" -d '{"input": [9]}' 127.0.0.1:1337/pipeline-sklearn/predict ; done

curl -X POST --header "Content-Type:application/json" -d '{"input": [9]}' 127.0.0.1:1337/pipeline-sklearn/predict & curl -X POST --header "Content-Type:application/json" -d '{"input": [11]}’ 127.0.0.1:1337/pipeline-sklearn/predict &
curl -X POST --header "Content-Type:application/json" -d '{"input": [12]}’ 127.0.0.1:1337/pipeline-sklearn/predict &
curl -X POST --header "Content-Type:application/json" -d '{"input": [7]}’ 127.0.0.1:1337/pipeline-sklearn/predict &
curl -X POST --header "Content-Type:application/json" -d '{"input": [8]}’ 127.0.0.1:1337/pipeline-sklearn/predict &
curl -X POST --header "Content-Type:application/json" -d '{"input": [15]}’ 127.0.0.1:1337/pipeline-sklearn/predict

time curl -X POST --header "Content-Type:application/json" -d '{"input": [9]}' 127.0.0.1:1337/pipeline-sklearn/predict & curl -X POST --header "Content-Type:application/json" -d '{"input": [10]}' 127.0.0.1:1337/pipeline-sklearn/predict & curl -X POST --header "Content-Type:application/json" -d '{"input": [6]}' 127.0.0.1:1337/pipeline-sklearn/predict & curl -X POST --header "Content-Type:application/json" -d '{"input": [18]}' 127.0.0.1:1337/pipeline-sklearn/predict & curl -X POST --header "Content-Type:application/json" -d '{"input": [2]}' 127.0.0.1:1337/pipeline-sklearn/predict & curl -X POST --header "Content-Type:application/json" -d '{"input": [15]}' 127.0.0.1:1337/pipeline-sklearn/predict

time curl -X POST --header "Content-Type:application/json" -d '{"input": [9]}' 127.0.0.1:1337/pipeline-sklearn/predict & curl -X POST --header "Content-Type:application/json" -d '{"input": [9]}' 127.0.0.1:1337/pipeline-sklearn/predict & curl -X POST --header "Content-Type:application/json" -d '{"input": [9]}' 127.0.0.1:1337/pipeline-sklearn/predict & curl -X POST --header "Content-Type:application/json" -d '{"input": [9]}' 127.0.0.1:1337/pipeline-sklearn/predict & curl -X POST --header "Content-Type:application/json" -d '{"input": [9]}' 127.0.0.1:1337/pipeline-sklearn/predict
