# admission-hook-example
This is a sample admission hook example

It will deny deployments based on your restricted(denylists) namespace name

The gke example is a basic one.
The aks example is a richer example which connects to appconfigmap to check allowed namespces

```
kubectl apply -f admission.yaml
kubectl create -f example-deployment.yaml -n simpler (This will work)
kubectl create -f example-deployment.yaml -n blockedns (This will fail)
```


To understand more :

Add the following as cloud function in gke or aks.

Cloud functions are simply lambda functions

Here in this code we have blocked **blockdns **  namespace from deploying.(Can be changed to run time fetch)

(Google cloud or GKE cluster)

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



###Aks cluster (Azure)
### The below code is a richer format which connects to azure appconfig where we can define key as namespace and value as either true or false

```
//Freezing namespace in azure

module.exports = async function (context, req) {
    const admissionRequest =  req.body;
    var reqns = admissionRequest.request.namespace;
    var isNamespaceFrozen="true"
    context.log('enigmaticcoder Received Request',req);
    const appConfig = require("@azure/app-configuration");
    const client = new appConfig.AppConfigurationClient(
    //we can pick this from env variable if needed
    'Endpoint=https://customcontrolconfig.azconfig.io;Id=7dmq-l0-s0:DWafE6gPCx0dhzPY2pj1;Secret=Sorrycantsharehere'
   );
    try{
      let configurationSetting = await client.getConfigurationSetting({key:reqns});
      context.log("This is the value of namespace from appconfig", configurationSetting.value);
      isNamespaceFrozen = configurationSetting.value
    }catch(err){
      context.log("No value is found in config map. Will freeze this namespace as value is set to ",isNamespaceFrozen);
    }
  context.log(`enigmaticcoder validating the deployment`);
    var admissionResponse = {
    apiVersion: "admission.k8s.io/v1",
    kind: "AdmissionReview",
    response: {
      uid: admissionRequest.request.uid,
      allowed: false,
      status: {
        "code": 403,
        "message": "You cannot do this because your namespace is frozen"
      }
    }
  }
  
  if ( isNamespaceFrozen == "true"){
    context.log("Namespace is frozen ",reqns);
  }else{
    admissionResponse = {
    apiVersion: "admission.k8s.io/v1",
    kind: "AdmissionReview",
    response: {
      uid: admissionRequest.request.uid,
      allowed: true,
    }
  }
  }
      
      context.res = {
          status: 200, /* Defaults to 200 */
          body: admissionResponse
   };
  }
```