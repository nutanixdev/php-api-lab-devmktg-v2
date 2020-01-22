Accessing The Nutanix REST APIs
###############################

To summarise really quickly, we currently have a Laravel application setup as follows.

- Base Laravel 6 application
- **AjaxController** (empty for now) and **HomeController** classes
- Customised routing to control which controller methods are used for each URL in the app
- Custom CSS stylesheets to control how our application looks
- Third-party libraries and packages to control additional appearance options and to provide additional JavaScript capabilities

That's all working properly, now.  The point of app, though, is to get information from a Nutanix Prism Central instance via REST API v3 and show that information inside our app.

Let's start doing that now.

The AjaxController class
........................

.. note::

   **Raw source file for this section, if required:** https://raw.githubusercontent.com/nutanixdev/lab-assets/master/php-lab-v2/AjaxController.php

So that the page isn't repeatedly reloaded every time we make a request, all Nutanix REST API requests will be completed via AJAX calls.  AJAX stands for Asynchronous JavaScript and XML and allows us to dynamically show information within the app without reloading the page.

Our **AjaxController.php** class is currently empty, but it is the core class used for all Asynchronous requests in our app.

For this reason, it does quite a few things.  Let's populate the contents of that class now.

#. Make sure a file named **AjaxController.php** exists in the **app/Http/Controllers** directory.  This file should exist as we created this class earlier.

#. The contents of the **AjaxController.php** file should be as follows:

    .. code-block:: php

        <?php

        namespace App\Http\Controllers;

        use App\ApiRequest;
        use App\ApiRequestParameters;

        use App\Http\Requests;

        class AjaxController extends Controller
        {

            /**
            * Load the dashboard's layout
            *
            * @return \Illuminate\Http\JsonResponse
            */
            public function postLoadLayout()
            {
                $source_json = json_decode(file_get_contents(base_path() . '/config/dashboard.json', true));
                return response()->json(['layout' => base64_decode($source_json->layout)]);
            }

            /**
            * Internal method used to generate a base64-encoded Gridster layout
            */
            public function getEncodeLayout()
            {
                $layout = '[{"col":1,"row":1,"size_x":1,"size_y":1},{"col":1,"row":2,"size_x":1,"size_y":1},{"col":1,"row":3,"size_x":1,"size_y":1},{"col":2,"row":1,"size_x":2,"size_y":1},{"id":"bigGraph","col":2,"row":2,"size_x":2,"size_y":2},{"col":4,"row":1,"size_x":1,"size_y":1},{"col":4,"row":2,"size_x":2,"size_y":1},{"col":4,"row":3,"size_x":1,"size_y":1},{"col":5,"row":1,"size_x":1,"size_y":1},{"col":5,"row":3,"size_x":1,"size_y":1},{"col":6,"row":1,"size_x":1,"size_y":1},{"col":6,"row":2,"size_x":1,"size_y":2},{"id":"footerWidget","col":1,"row":4,"size_x":6,"size_y":1}]';
                echo base64_encode($layout);
            }

            /**
            * Save the current layout to the JSON configuration file
            *
            * @return \Illuminate\Http\JsonResponse
            */
            public function postSaveToJson()
            {
                $layout = base64_encode($_POST['_serialized']);
                $fullJson = json_encode(['version' => '1.0', 'layout' => $layout]);
                $file = base_path() . '/config/dashboard.json';
                file_put_contents($file, $fullJson);
                return response()->json(['result' => 0]);
            }

            /**
            * Return the dashboard to the default, before any changes are made
            *
            * @return \Illuminate\Http\JsonResponse
            */
            public function postLoadDefault()
            {
                $layout = file_get_contents(base_path() . '/resources/install/dashboard-default.json');
                return response()->json(['layout' => $layout]);
            }

            /**
            * Return a list of Prism Central managed entities, based on a specified entity identifier/name
            *
            * @return \Illuminate\Http\JsonResponse
            */
            public function postPcListEntities()
            {

                $entity = $_POST['_entity'];

                $body = [ 'kind' => $entity ];

                $parameters = ['username' => $_POST['_username'], 'password' => $_POST['_password'], 'cvmAddress' => $_POST['_cvmAddress'], 'objectPath' => $entity . 's/list', 'method' => 'POST', 'body' => json_encode($body), 'entity' => $entity ];
                
                $results = (new ApiRequest(new ApiRequestParameters($parameters)))->doApiRequest(null, 'POST');

                return response()->json(['results' => $results]);
            }

            /**
            * 
            * Return some high level storage container performance stats
            *
            * @return \Illuminate\Http\JsonResponse
            */
            public function postContainerInfo()
            {

                /**
                * sample request shown below
                * 
                * https://{cvm_ip}:9440/PrismGateway/services/rest/v1/containers/{container_uuid}/stats/?metrics={metric}&startTimeInUsecs={start_time}&endTimeInUsecs={end_time}&interval={interval_in_secs}
                * 
                * Metric used below is controller_avg_io_latency_usecs
                * 
                */

                $parameters = ['username' => $_POST['_username'], 'password' => $_POST['_password'], 'cvmAddress' => $_POST['_cvmAddress'], 'objectPath' => 'storage_containers'];

                $containers = (new ApiRequest(new ApiRequestParameters($parameters)))->doApiRequest();

                $firstContainerId = $containers->entities[0]->id;
                $containerCount = $containers->metadata->grand_total_entities;

                $parameters = [
                    'username' => $_POST['_username'],
                    'password' => $_POST['_password'],
                    'topLevelPath' => 'PrismGateway/services/rest/v1',
                    'objectPath' => 'containers',
                    'objectId' => $firstContainerId,
                    'objectSubPath' => 'stats',
                    'metric' => 'controller_avg_io_latency_usecs',
                    'cvmAddress' => $_POST['_cvmAddress'],
                    'cvmPort' => '9440',
                    'connectionTimeout' => 3,
                    'method' => 'GET',
                    'startTime' => \Carbon\Carbon::now()->subHour(4),
                    'endTime' => \Carbon\Carbon::now(),
                    'interval' => 30
                ];

                $results = (new ApiRequest(new ApiRequestParameters($parameters)))->doApiRequest();

                return response()->json(['containerCount' => $containerCount, 'stats' => [$results->statsSpecificResponses[0]->values]]);
            }
        }

What does the **AjaxController** class do?

- Contains various methods to load the initial dashboard layout and revert the layout to a sensible default, if required.  These methods are **postLoadLayout()** and **postLoadDefault()**.
- Contains methods to save the user's layout, if they decide to keep their changes.  This method, **postSaveToJson()**, is called when the user clicks the **Save Layout** button in the application and instructs the app how to save the customised layout for later use.
- Contains the most important method: **postPcListEntities()**.  This method is used in all our API requests and accepts a number of parameters to specify how the request should work.
- Lastly the method named **postContainerInfo()** has been left in the **AjaxControllerClass** class.  This isn't used in v2 of this lab but can be used for reference should you decide to add stats later.

Creating the JavaScript
.......................

With our classes created, the final (and probably most important) file can be created.

However, because this is such a key part of the application, it will be broken out into a separate (and final) part of this lab.

Click the button to continue!