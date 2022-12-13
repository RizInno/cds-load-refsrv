# Reference Server
This server acts as reference implementation and baseline for the client side test tool. 

The tool provides a simple test server to evaluate the varying settings. To prepare the server, execute the following:
1. Clone the repository to your machine
2. Execute the dependency installation in the main directory `npm i`
3. Either run the server via `cds watch` or build and deploy it to the SAP BTP.



## Mass Change Service
To optimize the performance of propagating mass changes (in my case merely creates) from client to server, the server implements a special service. This service is called `MassChangeService` and is located in the `srv` folder. The service is a simple wrapper around the standard `ReferenceService` and provides a generic action `insertAll` to insert multiple entities at once. The service is defined as follows:
```cds
@protocol: 'rest'
service MassChangeService {
    
    @open
    type AnyArray {};
    
    action insertAll(insEntity : String, insArray: AnyArray) returns AnyArray;
 
}
```
The implementation of the service is then handled in [./srv/handlers/mass.js](./srv/handlers/mass.js).



### REST Adapter content size limitation
The standard CAP REST adapter uses the limitation from the underlying 'express' framework. That means, that the maximum size of a request is 100kb. If you want to increase this limit, you currently must patch the `@sap/cds-rest/libx/rest/RestAdapter.js` file and change the following
```diff
-  router.use(express.json()) // REVISIT: -> belongs to the parses
+
+  
+  console.log('MST - REST Adapter Patch: Set limit to 50mb')
+  router.use(express.json({limit:'50mb'})) // REVISIT: -> belongs to the parses
+  //router.use(express.json()) // REVISIT: -> belongs to the parses
```

You can use the npm package `patch-package` to patch the file automatically.

When you run patch-package with the latest version of npm, you will get an error message. To fix this, you can use the following set of commands:
1. delete `package-lock.json`
2. `npm i --lockfile-version 3`
3. `npx patch-package --patch-dir ./patches @sap/cds`
4. delete `package-lock.json` one more time
5. `npm i`

