Webapp
=======

General
-------

The webapp is a web interface for viewing, planing, assigning and exporting routes. It is based on Node.js and uses the Mapbox GL JS library for rendering the map and the Mapbox Optimization v2 API for route calculation.
Authentication is done via AWS Cognito. Through this, we are not responsible for the security of the user data.
For design we are using Bootstrap 5.3.0-alpha2. The webapp is hosted on AWS Elastic Beanstalk. The webapp is currently ready for use in a production environment.


Infrastructure
--------------

The webapp is based on a `Node.js <https://nodejs.org/en/>`_ server. The server is responsible for the communication with the database and the communication with the clients. The server is also responsible for the authentication and authorization of the users.

It is hosted inside Elastic Beanstalk on AWS.


.. image:: images/ElasticBeanstalk-Flowchart.png
    :width: 500
    :alt: Topology of the webapp infrastructure on AWS


It will be extended in the future with two availability zones and a Elastic Load Balancer (ELB) for load balancing and high availability.

Design
------

The design of the webapp is based on Bootstrap 5.3.0-alpha2. Bootstrap is a free and open-source CSS framework directed at responsive, mobile-first front-end web development. It contains CSS- and JavaScript-based design templates for typography, forms, buttons, navigation, and other interface components.

With the help of Bootstrap we can create a responsive design that works on every device. Because Bootstrap is a very popular framework the user is already familiar with the design and can use the webapp without any problems. Due to the sleek, modern design, the webapp is visually appealing and easy to use.

.. image:: images/Screenshot_AssignSites.png
    :width: 500
    :alt: Screenshot of the webapp

Mapview
--------

The Mapview is intended to be a simple, easy to use, and easy to understand web interface for viewing, planing, assigning and exporting routes.

It is based on Node.js and uses the `Mapbox GL JS <https://docs.mapbox.com/mapbox-gl-js/guides/>`_ library for rendering the map and the `Mapbox Optimization v2 API <https://docs.mapbox.com/api/navigation/optimization/>`_ for route calculation.
The routes can be exported as a ICS file for import into a calendar application. With this way you can use the calendar application of your choice for navigation. Cars (like Tesla) with a calendar integration can also use the calendar for navigation.

The workflow uses a 3 step process:

1. Select the waypoints on the map ba drawing a polygon around them.
2. Generate the route with the Mapbox Optimization v2 API.
3. See the stops and download the ICS file.


Mapbox
^^^^^^^

Why Mapbox?
''''''''''''

For short: Mapbox has a lot of free requests per month and it has more features than Google Maps.

The longer version: Google Maps is the most used map service and it is very good. But it has some disadvantages. A Google Maps object itself (just the map without any id) is free no mather how many loads the map has. But thats where the fun is only starting.
Everything else is billed via the Google Maps Platform. You can try to calculate how much you would pay via the `calculator <https://mapsplatform.google.com/pricing/>`_. There you can see that you "only" have $200 free monthly usage.
For a student project this is a sufficient contingent of free requests. But think a little bit bigger: 28.500 loads only for one SDK is not enough if you want to sell the product. And you also have to calculate the other features like the Geocoding API or the Route API.
So things can get very expensive very fast.

This is where Mapbox comes in. After Google raised the prices for Maps companies switched to Mapbox. And Mapbox has a few advantages over Google Maps. The biggest one is that you get 50.000 free loads per month on the JavaScript SDK alone.
Every API is billed separately and the amount of free requests is significantly higher than Google Maps.
For AirTrack we are also using the `Mapbox Optimization v2 API <https://docs.mapbox.com/api/navigation/optimization/>`_ witch is currently in public beta and therefore free to use. This API is used for calculating routes with a (theoretically) 1000 given waypoints.
It solves the so called "Traveling Salesman Problem" (TSP). Google Maps does not have a similar API. You can find more about the TSP `here <https://en.wikipedia.org/wiki/Travelling_salesman_problem>`_.


Mapbox vs. Google Maps Platform
''''''''''''''''''''''''''''''''

This comparison is based on what we need for AirTrack. Based on this we decided to use Mapbox.




+------------------------------------------------+----------------------------------------+----------------------+
| Features                                       | Mapbox                                 | Google Maps Platform |
+================================================+========================================+======================+
| JavaScript SDK free loads per month            | 50.000                                 | 28,500               |
+------------------------------------------------+----------------------------------------+----------------------+
|| JavaScript SDK $ per 1000 loads per month     || $5 (50.001-100.000 loads per month)   || $7                  |
||                                               || $4 (100.001-200.000 loads per month)  ||                     |
||                                               || $3(200.001-1.000.000 loads per month) ||                     |
+------------------------------------------------+----------------------------------------+----------------------+
| Geocoding API free requests per month          | 100.000                                | 28,500               |
+------------------------------------------------+----------------------------------------+----------------------+
| Route Optimization API free requests per month | Unlimited (because of public beta)     | No API               |
+------------------------------------------------+----------------------------------------+----------------------+
| API & SDK requests billed separately           | Yes                                    | No                   |
+------------------------------------------------+----------------------------------------+----------------------+
| Detailed documentation                         | Yes                                    | Yes                  |
+------------------------------------------------+----------------------------------------+----------------------+
| Support                                        | Only in paid version                   | Only in paid version |
+------------------------------------------------+----------------------------------------+----------------------+
| Custom map styles                              | Yes                                    | Yes                  |
+------------------------------------------------+----------------------------------------+----------------------+
| Offline maps                                   | Yes                                    | No                   |
+------------------------------------------------+----------------------------------------+----------------------+
| Possibility for self-hosting                   | Yes (Mapbox Atlas)                     | No                   |
+------------------------------------------------+----------------------------------------+----------------------+
| Street view                                    | No                                     | Yes                  |
+------------------------------------------------+----------------------------------------+----------------------+
| Easy implementation of dynamic map changes     | Yes                                    | No                   |
+------------------------------------------------+----------------------------------------+----------------------+
| Turf.js integration                            | Yes                                    | No                   |
+------------------------------------------------+----------------------------------------+----------------------+
| **Open Source**                                | **Yes**                                | No                   |
+------------------------------------------------+----------------------------------------+----------------------+

Route Optimization
^^^^^^^^^^^^^^^^^^

What is Mapbox Optimization v2 API?
'''''''''''''''''''''''''''''''''''
It is a beta version of the Mapbox Optimization API. It is currently free to use.

Mapbox defines it as follows:

The Optimization v2 API is an API for calculating efficient plans for vehicles to visit multiple locations. These are commonly known as vehicle routing problems.

The Optimization v2 API enables you to submit vehicle routing problems in the form of routing problem documents describing the number of vehicles in your fleet, locations to be visited, and other constraints that are relevant to your real-world problem. The API returns an optimized solution for your routing problem as a solution document that describes a route plan for each vehicle.

Source: `Mapbox Optimization v2 API <https://docs.mapbox.com/api/navigation/optimization/>`_

What can it do?
''''''''''''''''

If you want to get a optimized route you can send them with a POST request a JSON file with the following information:

.. code-block:: json

    POST /optimized-trips/v2?access_token=TOKEN HTTP/1.1
    Host: api.mapbox.com
    Content-Type: application/json
    { 
        "version": 1,
        "vehicles": [...],
        "services": [...]
    }

This returns a route ID. This ID can be used to get the status or the optimized route itself.

The input supports the following features:

.. code-block:: json

    {
        "version": 1,
        "locations": [...],
        "vehicles": [...],
        "services": [...],
        "shipments": [...]
    }


To get the status of all routes you can send a GET request:

.. code-block:: json

    GET /optimized-trips/v2?access_token=TOKEN
    Content-Type: application/json
    Host: api.mapbox.com

If the status is "complete" you can get the route with the following GET request:

.. code-block:: json

    GET /optimized-trips/v2/{route-id}?access_token=TOKEN
    Content-Type: application/json
    Host: api.mapbox.com


How are we using it?
''''''''''''''''''''

We are using it as it is described above. We are sending a JSON file with the waypoints to the API and get a route ID back.
Because we only have a maximum of 10 waypoints we can wait 15 seconds and then the route optimization is complete. We then get the route with the route ID and display the stops in the Mapview.
When you want to export the route as a ICS file we send the route ID to the ICS export function. The ICS export function then gets the route with the route ID and creates a ICS file with the stops.

More on the ICS export function can be found below section.


ICS Export
^^^^^^^^^^

The ICS export function is a API call to AWS Lambda. It is based on a FastAPI application. The python file is using the iCalender library to create the ICS file.

This call is made at the last step of the Mapview. There the user can see the individual stops and download the ICS file. The ICS file can then be imported into a calendar application of your choice. With this way you can use the calendar application for navigation. Cars (like Tesla) with a calendar integration can also use the calendar for navigation.

How does it work?
''''''''''''''''''

.. image:: images/Get-ICS-file.png
    :width: 500
    :alt: Flowchart of the ICS export function

When the button "Export as ICS" is clicked, the route ID of the optimized route is sent to the API. The AWS API Gateway forwards the request to the FastAPI service in a AWS Lambda function. 
The FastAPI service then gets the route ID and makes a GET request to the Mapbox optimization v2 API to get the optimized route. The response is a JSON file with the route information.
After that the FastAPI service then parses the JSON file and creates a ICS file with the iCalender library. The ICS file is then returned to the user and is downloaded automatically.

You can find out more information about this in the `API documentation <https://airtrack.readthedocs.io/en/latest/api.html>`_.

AssetView
^^^^^^^^^

.. image:: images/Screenshot_AssignSites.png
    :width: 500
    :alt: Screenshot of the AssetView

The assigned sites view is a view where the user can see his assigned sites in a list. 

This list shows the following information:

* Site ID
* Address
* Access Instructions
* Access Restrictions
* Special Instructions

Gathering the data is rather easy. We are using a GET request to the API to get the assigned sites. The API returns a JSON object with the assigned sites. We then return the JSON object to the AssetView and display the information in a list.

.. code-block:: javascript

    app.get('/get-sites-of-user', isAuthenticated, (req, res) => {
      let user = req.session.userName;
      //get the sites of the user
      const response = fetch('https://98j8m82ij0.execute-api.eu-central-1.amazonaws.com/production/sites-by-user/' + user)
          .then(response => response.json())
          .then(data => {
            //return the sites
            res.send(data);
          })
          .catch(error => console.error(error));
    });


Admin Dashboard
^^^^^^^^^^^^^^^

.. image:: images/Screenshot_AdminDashboard.png
    :width: 500
    :alt: Screenshot of the Admin Dashboard

If you have the privilege to access the admin dashboard you can access the following features:

* See your assigned sites on the map
* See the assigned sites in a list
* Create a new user
* Assign sites to a user
* Show all users

Mapview and AssetView is the same as described above. Every user can access the Mapview and the AssetView.

Creating a new user
'''''''''''''''''''

.. image:: images/Screenshot_CreateUser.png
    :width: 500
    :alt: Screenshot of the Create User form

To create a new user you have to fill out the form. Most notably the role assignment is important. If you want to create a user with admin privileges you have to select "Admin" in the role assignment dropdown. We are advising to create a new user with user privileges.



Assigning sites
^^^^^^^^^^^^^^^

.. image:: images/Screenshot_AssignSitesToUser.png
    :width: 500
    :alt: Screenshot of the Assign Sites form

To assign sites to a user you have to select the user in the dropdown and then select the sites you want to assign to the user. You can select multiple sites at once. After you have selected the sites you can click on the "Apply" button to assign the sites to the user.

Show all users
^^^^^^^^^^^^^^

.. image:: image/Screenshot_UserList.png
    :width: 500
    :alt: Screenshot of the Show All Users form

Here are all users displayed in a list. You can see the following information:

* Username
* Email
* Role
* Delete (Button)

The most prominent feature is the delete button. If you click on the delete button the user is deleted from the database. This is a permanent deletion and can not be undone.
If you delete a user by accident you have to create a new user with the same username and email address, let the user confirm the account and then assign the sites to the user again.



.. End