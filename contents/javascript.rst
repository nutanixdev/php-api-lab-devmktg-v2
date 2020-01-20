Creating the JavaScript
#######################

The final and probably most important file is **ntnx.js**.  For our app, it could be looked at as the interface between the user's browser and the **AjaxController.php** class created in the previous section.

While it's true that JavaScript can be used for a server-side functions via Node.js/NPM plus many others, we aren't using that in our app.

To ensure everything is modular and easy to modify later, the **ntnx.js** JavaScript has been written in a way that allows it to be easy to read.

#. Make sure a file named **ntnx.js** exists in the **public/js** directory.  This file should exist as we created this class earlier.

#. The contents of the **ntnx.js** file should be as follows:

    .. note::

      **Raw source file for this section, if required:** https://raw.githubusercontent.com/nutanixdev/lab-assets/master/php-lab-v2/ntnx.js

    .. raw:: html

       <strong><font color="red">Important note: The ntnx.js file is reasonably long.  Please create it first, then we'll look at what it does.</font></strong><br>

    .. code-block:: JavaScript

        var NtnxDashboard;
        NtnxDashboard = {

            /**
            * 
            * @param {*} config 
            * 
            * Initialise the application
            * 
            */
            init: function ( config )
            {
                this.config = config;
                this.setupGridster();
                this.setUI();
                this.bindEvents();

                /* load the saved/default dashboard when the DOM is ready */
                $( document).ready( function() {

                    NtnxDashboard.loadLayout();

                });

            },
            /* init */

            /**
            * 
            * @param {*} cell 
            * 
            * Remove existing contents of a specified DOM element
            * 
            */
            resetCell: function( cell )
            {
                $( '#' + cell ).html( '<span class="gs-resize-handle gs-resize-handle-both"></span>' );
            },
            /* resetCell */

            /**
            * 
            * @param {*} token 
            * @param {*} cvmAddress 
            * @param {*} username 
            * @param {*} password 
            * @param {*} entity 
            * @param {*} pageElement 
            * @param {*} elementTitle 
            * 
            * main function to build and send the entity list requests
            * the previous version of this used a single function for each request
            * 
            */
            pcListEntities: function( token, cvmAddress, username, password, entity, pageElement, elementTitle ) {

                pcEntityInfo = $.ajax({
                    url: '/ajax/pc-list-entities',
                    type: 'POST',
                    dataType: 'json',
                    data: { _token: token, _cvmAddress: cvmAddress, _username: username, _password: password, _entity: entity, _pageElement: pageElement, _elementTitle: elementTitle },
                });

                pcEntityInfo.done( function(data) {

                    NtnxDashboard.resetCell( pageElement );
                    $( '#' + pageElement  ).addClass( 'info_big' ).append( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">' + elementTitle + '</div><div>' + data.results.metadata.total_matches + '</div><div></div>');

                    switch( entity ) {
                        case 'project':

                            $( '#project_details' ).addClass( 'info_big' ).html( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">Project List</div>' );

                            $( data.results.entities ).each( function( index, item ) {
                                $( '#project_details' ).append( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">' +  item.status.name + '</div>' );
                            });

                            $( '#project_details' ).append( '</div><div></div>' );

                        default:
                            break;
                    }

                });

            },
            /* pcListEntities */

            /**
            * 
            * @param {*} token 
            * 
            * Remove the big graph DOM element from the page entirely
            * Legacy function from previous version, but may be used again
            * 
            */
            removeGraph: function( token ) {
                var gridster = $( '.gridster ul' ).gridster().data( 'gridster' );
                var element = $( '#bigGraph' );
                gridster.remove_widget( element );
            },
            /* removeGraph */

            /**
            * 
            * @param {*} token 
            * 
            * Revert the altered grid layout to the default from when the lab app was built
            * 
            */
            restoreDefaultLayout: function( token ) {
                var gridster = $( '.gridster ul' ).gridster().data( 'gridster' );
                gridster.remove_all_widgets();

                /* AJAX call to get the default layout from the system's default dashboard */
                request = $.ajax({
                    url: '/ajax/load-default',
                    type: 'POST',
                    dataType: 'json',
                    data: { _token: token },
                });

                request.done( function(data) {
                    serialization = Gridster.sort_by_row_and_col_asc( JSON.parse( data.layout ) );
                    $.each( serialization, function() {
                        gridster.add_widget('<li id="' + this.id + '" />', this.size_x, this.size_y, this.col, this.row);
                    });

                    NtnxDashboard.resetCell( 'footerWidget' );
                    $( 'li#footerWidget' ).addClass( 'panel' ).append( '<div class="panel-body"><div id="controllerIOPS" style="height: 150px; width: 1000px; text-align: center;"></div></div>' );
                    $( '#status_new' ).html( 'Default layout restored. Don\'t forget to save!' ).removeClass().addClass( 'alert' ).addClass( 'alert-warning' ).slideDown( 300 );
                });

                request.fail(function ( jqXHR, textStatus, errorThrown )
                {
                    $( '#status_new' ).removeClass().html( textStatus + ' - ' + errorThrown ).addClass( 'alert' ).addClass( 'alert-error' );
                });

            },
            /* restoreDefaultLayout */

            /**
            * 
            * @param {*} token 
            * 
            * Get the grid's layout and serialize it in a format appropriate for transmission
            * 
            */
            serializeLayout: function( token ) {
                var gridster = $( '.gridster ul' ).gridster().data( 'gridster' );
                var json = gridster.serialize();
                $( '#serialized' ).html( JSON.stringify( json ) );
            },
            /* serializeLayout */

            /**
            * 
            * @param {*} token 
            * 
            * Save the user's layout changes to on-disk JSON file
            * 
            */
            saveLayout: function( token ) {
                /* get the gridster object */
                var gridster = $( '.gridster ul' ).gridster().data( 'gridster' );
                /* serialize the current layout */
                var json = gridster.serialize();

                /* convert the layout to json */
                var serialized = JSON.stringify( json );

                /* AJAX call to save the layout the app's configuration file */
                request = $.ajax({
                    url: '/ajax/save-to-json',
                    type: 'POST',
                    dataType: 'json',
                    data: { _token: token, _serialized: serialized },
                });

                request.done( function(data) {
                    $( '#status_new' ).removeClass().html( 'Dashboard saved!' ).addClass( 'alert' ).addClass( 'alert-success' ).slideDown( 300 ).delay( 2000 ).slideUp( 300 );
                });

                request.fail(function ( jqXHR, textStatus, errorThrown )
                {
                    $( '#status_new' ).removeClass().html( textStatus + ' - ' + errorThrown ).addClass( 'alert' ).addClass( 'alert-error' );
                });

            },
            /* saveLayout */

            /**
            * 
            * Can't remember what this is for lol
            * Just kidding - it's for some tests carried out during development
            * 
            */
            s4: function()
            {
                return Math.floor((1 + Math.random()) * 0x10000).toString(16).substring(1);
            },
            /* s4 */

            /**
            * 
            * Load the existing/saved grid layout from dashboard.json
            * This file holds the default layout if no changes have been made, or the layout setup by the user after saving
            * 
            */
            loadLayout: function()
            {
                request = $.ajax({
                    url: '/ajax/load-layout',
                    type: 'POST',
                    dataType: 'json',
                    data: {},
                });

                var cvmAddress = $( '#cvmAddress' ).val();
                var username = $( '#username' ).val();
                var password = $( '#password' ).val();

                request.done( function( data ) {
                    var gridster = $( '.gridster ul' ).gridster().data( 'gridster' );
                    var serialization = JSON.parse( data.layout );

                    serialization = Gridster.sort_by_row_and_col_asc(serialization);
                    $.each( serialization, function() {
                        gridster.add_widget('<li id="' + this.id + '" />', this.size_x, this.size_y, this.col, this.row);
                    });

                    /* add the chart markup to the largest containers */
                    $( 'li#footerWidget' ).addClass( 'panel' ).append( '<div class="panel-body"><div id="controllerIOPS" style="height: 150px; width: 1000px; text-align: center;"></div></div>' );

                    NtnxDashboard.resetCell( 'bigGraph' );
                    $( '#bigGraph' ).addClass( 'info_hilite' ).append( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">Hey ...</div><div>Enter your Prism Central details above, then click the Go button ...</div>');
                    $( '#hints' ).addClass( 'info_hilite' ).append( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">Also ...</div><div>Drag &amp; Drop<br>The Boxes</div>');

                });

                request.fail(function ( jqXHR, textStatus, errorThrown )
                {
                    /* Display an error message */
                    alert( 'Unfortunately an error occurred while processing the request.  Status: ' + textStatus + ', Error Thrown: ' + errorThrown );
                });
            },
            /* loadLayout */

            /**
            * 
            * Setup the page's main grid
            * 
            */
            setupGridster: function ()
            {
                $( function ()
                {

                    var gridster = $( '.gridster ul' ).gridster( {
                        widget_margins: [ 10, 10 ],
                        widget_base_dimensions: [ 170, 170 ],
                        max_cols: 10,
                        autogrow_cols: true,
                        resize: {
                            enabled: true
                        },
                        draggable: {
                            stop: function( e, ui, $widget ) {
                                $( '#status_new' ).html( 'Your dashboard layout has changed. Don\'t forget to save!' ).removeClass().addClass( 'alert' ).addClass( 'alert-warning' ).slideDown( 300 );
                            }
                        },
                        serialize_params: function ($w, wgd) {

                            return {
                                /* add element ID to data*/
                                id: $w.attr('id'),
                                /* defaults */
                                col: wgd.col,
                                row: wgd.row,
                                size_x: wgd.size_x,
                                size_y: wgd.size_y
                            }

                        }
                    } ).data( 'gridster' );

                } );
            },
            /* setupGridster */

            /**
            * 
            * Apply tooltips to various elements and setup the delay on some animations
            * 
            */
            setUI: function ()
            {

                $( 'div.alert-success' ).delay( 3000 ).slideUp( 1000 );
                $( 'div.alert-info' ).delay( 3000 ).slideUp( 1000 );

                $(function () {
                    $('[data-toggle="tooltip"]').tooltip()
                })

            },
            /* setUI */

            /**
            * 
            * Bind events that will get triggered in response to various actions
            * In particular, button clicks
            * 
            */
            bindEvents: function()
            {

                var self = NtnxDashboard;

                $( '#goButton' ).on( 'click', function ( e ) {

                    var cvmAddress = $( '#cvmAddress' ).val();
                    var username = $( '#username' ).val();
                    var password = $( '#password' ).val();

                    if( ( cvmAddress == '' ) || ( username == '' ) || ( password == '' ) )
                    {
                        NtnxDashboard.resetCell( 'bigGraph' );
                        $( '#bigGraph' ).addClass( 'info_error' ).append( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">Awww ...</div><div>Did you forget to enter something?</div>');
                    }
                    else
                    {
                        NtnxDashboard.resetCell( 'bigGraph' );
                        $( '#bigGraph' ).html( '<span class="gs-resize-handle gs-resize-handle-both"></span>' ).removeClass( 'info_hilite' ).removeClass( 'info_error' ).addClass( 'info_big' ).append( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">Ok ...</div><div>Gathering environment details ...</div>');
                        NtnxDashboard.resetCell( 'hints' );
                        $( '#hints' ).html( '<span class="gs-resize-handle gs-resize-handle-both"></span>' ).addClass( 'info_hilite' ).append( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">Also ...</div><div>Drag &amp; Drop<br>The Boxes</div>');
                    
                        NtnxDashboard.pcListEntities( $( '#csrf_token' ).val(), cvmAddress, username, password, 'cluster', 'registered_clusters', 'Registered Clusters' );
                        NtnxDashboard.pcListEntities( $( '#csrf_token' ).val(), cvmAddress, username, password, 'image', 'image_count', 'Images' );
                        NtnxDashboard.pcListEntities( $( '#csrf_token' ).val(), cvmAddress, username, password, 'vm', 'vm_count', 'VMs' );
                        NtnxDashboard.pcListEntities( $( '#csrf_token' ).val(), cvmAddress, username, password, 'host', 'host_count', 'Hosts &amp; PC Nodes' );
                        NtnxDashboard.pcListEntities( $( '#csrf_token' ).val(), cvmAddress, username, password, 'project', 'project_count', 'Project Count' );
                        NtnxDashboard.pcListEntities( $( '#csrf_token' ).val(), cvmAddress, username, password, 'app', 'app_count', 'Calm' );
                    }

                    e.preventDefault();
                });

                $( '.serializeLayout' ).on( 'click', function( e ) {
                    NtnxDashboard.serializeLayout( $( '#csrf_token' ).val() );
                    e.preventDefault();
                });

                $( '.saveLayout' ).on( 'click', function( e ) {
                    NtnxDashboard.saveLayout( $( '#csrf_token' ).val() );
                    e.preventDefault();
                });

                $( '.defaultLayout' ).on( 'click', function( e ) {
                    NtnxDashboard.restoreDefaultLayout( $( '#csrf_token' ).val() );
                    e.preventDefault();
                });

                $( '.removeGraph' ).on( 'click', function( e ) {
                    NtnxDashboard.removeGraph( $( '#csrf_token' ).val() );
                    e.preventDefault();
                });

                $( '.containerStats' ).on( 'click', function( e ) {
                    NtnxDashboard.containerInfo( $( '#csrf_token' ).val(), $( '#cvmAddress' ).val(), $( '#username' ).val(), $( '#password' ).val() );
                    e.preventDefault();
                });

                $( '.testButton' ).on( 'click', function( e ) {
                    $( '#registered_clusters' ).html( 'Hello' );
                    e.preventDefault();
                });

            },
            /* bindEvents */

        };

        NtnxDashboard.init({

        });

What does the **ntnx.js** script do?  The functions of **ntnx.ns**, in load-time order, are as follows.

1. Initialises the user interface via the **init** function.
2. Creates an instance of the jQuery **gridster** plugin class and configures the properties of that instance.  For our app, we are setting things like the element margins, the number of columns and telling the elements they are "draggable".
3. Altering a small number of UI elements so they appear correctly.
4. Binding the user interface events to other functions within **ntnx.js**.  This is a critical step as it instructs the browser and the JavaScript what to do when "something" happens.  For example, which part of the script should execute when a user enters cluster info and clicks the "Go!" button?

You'll note that many of the functions in **ntnx.js** are "mirrored" by methods in the **AjaxController.php** class.  This is very much by design as click events in our JavaScript have been written to call the request methods in **AjaxController.php**.  It's worth noting that jQuery could easily complete the AJAX requests itself without the need for **AjaxController.php**, but the app has been writtin this way to demonstrate how to route requests through to the **AJaxController.php** methods.

Loading the UI
..............

This final load-time action has been split into its own small section as it essentially controls what the user sees upon loading the app.

1. An AJAX POST request is made to the **/ajax/load-layout** PHP method.
2. The **/ajax/load-layout** request loads the saved layout from the **/config/dashboard.json** file we created earlier.
3. The contents of **/config/dashboard.json** are parsed and the individual UI elements ("boxes") are created.  Remember that the contents 
4. Finally, CSS classes are added to the new UI elements, e.g. setting background colour and font-size.

JavaScript functions
....................

The other functions within **ntnx.js** are only executed when specific events are fired.  Let's look at the **pcListEntities** function in more detail now.

The **pcListEntities** function is as follows.  Please don't add this code again; it is already part of your **ntnx.js** script.

.. code-block:: JavaScript

    /**
     * 
     * @param {*} token 
     * @param {*} cvmAddress 
     * @param {*} username 
     * @param {*} password 
     * @param {*} entity 
     * @param {*} pageElement 
     * @param {*} elementTitle 
     * 
     * main function to build and send the entity list requests
     * the previous version of this used a single function for each request
     * 
     */
    pcListEntities: function( token, cvmAddress, username, password, entity, pageElement, elementTitle ) {

        pcEntityInfo = $.ajax({
            url: '/ajax/pc-list-entities',
            type: 'POST',
            dataType: 'json',
            data: { _token: token, _cvmAddress: cvmAddress, _username: username, _password: password, _entity: entity, _pageElement: pageElement, _elementTitle: elementTitle },
        });

        pcEntityInfo.done( function(data) {

            NtnxDashboard.resetCell( pageElement );
            $( '#' + pageElement  ).addClass( 'info_big' ).append( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">' + elementTitle + '</div><div>' + data.results.metadata.total_matches + '</div><div></div>');

            switch( entity ) {
                case 'project':

                    $( '#project_details' ).addClass( 'info_big' ).html( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">Project List</div>' );

                    $( data.results.entities ).each( function( index, item ) {
                        $( '#project_details' ).append( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">' +  item.status.name + '</div>' );
                    });

                    $( '#project_details' ).append( '</div><div></div>' );

                default:
                    break;
            }

        });

    },
    /* pcListEntities */

Going through this function, we can see it does the following things.

1. An AJAX POST request is made to the **/ajax/pc-list-entities** PHP method (we'll also look at that shortly).
2. If the request was successful, the results of the AJAX request are parsed.
3. The parsed data is dynamically shown in the app UI via the jQuery **.append** method.
4. The location and formatting of the information shown in the browser is controlled via the **entity**, **pageElement** and **elementTitle** parameters.

From **app/Http/Controllers/AjaxController.php**, the **/ajax/pc-list-entities** method, **postPcListEntities**, is as follows.

.. code-block:: php

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

In this example, **postPcListEntities** is called via an HTTP POST request to **pc-list-entities**.

Going through this method, we can see it does the following things.

1. Gets the requests entity name from the **_entity** POST variable.
2. Sets up a very simple HTTP POST request body that tells the v3 API which "kind" of entity to return.
3. Creates an array containing a number of variables e.g. cluster username & password, the IP address of the cluster or CVM and the API endpoint we want to query.  This array is used as the request parameters shortly.
4. An instance of our **ApiRequest** class is created, with an instance of our **ApiRequestParameters** class passed to the **ApiRequest** constructor.
5. Using method-chaining (**->** in PHP), we are then calling **doApiRequest** to execute the actual request.
6. Returns the results of the API request's JSON response and makes the entire response available to the JavaScript function that called it.

Final Testing
.............

With all our classes, JavaScript, views and styling files in place, the app should now be ready to test!

#. Ensure your local web server is running by running the following command, if it isn't already running.

   .. code-block:: bash

      php artisan serve

#. If using the default port, browse to **http://127.0.0.1:8000**.

#. If everything has been setup correctly, you'll see a collection of UI elements ("boxes") displayed on the screen, with fields at the top for cluster/CVM IP address, username and password.

   .. figure:: images/ui_loaded.png

#. Enter your cluster/CVM IP address, username and password, then click the "Go!" button.

#. If everything has been setup correctly, you should see the app load as shown below.

   .. figure:: images/request_completed.png