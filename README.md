# admission-hook-example
This is a sample admission hook example



kubectl apply -f admission.yaml
kubectl create -f example-deployment.yaml -n simpler (This will work)
kubectl create -f example-deployment.yaml -n blockedns (This will fail)



To understand more :

Add the following as cloud function in gke or aks.

Cloud functions are simply lambda functions

Here in this code we have blocked **blockdns **  namespace from deploying.(Can be changed to run time fetch)

```
/**
 * Responds to any HTTP request.
 *
 * @param {!express:Request} req HTTP request context.
 * @param {!express:Response} res HTTP response context.
 */
'use strict';

exports.denyns = function denyns (req, res) {
  console.log("enigmaticcoder Received Request");
  console.log('enigmaticcoder',req);
  var admissionRequest = req.body;

  // Get a reference to the pod spec
  var reqns = admissionRequest.request.namespace;

  console.log(`enigmaticcoder validating the deployment`);
  var admissionResponse = {
  apiVersion: "admission.k8s.io/v1",
  kind: "AdmissionReview",
  response: {
    uid: admissionRequest.request.uid,
    allowed: false
  }
}


if (reqns == "blockedns"){
  //do nothing
}else{

  admissionResponse = {
  apiVersion: "admission.k8s.io/v1",
  kind: "AdmissionReview",
  response: {
    uid: admissionRequest.request.uid,
    allowed: true
  }
}

}


  res.setHeader('Content-Type', 'application/json');
  res.send(JSON.stringify(admissionResponse));
  res.status(200).end();
};
```

