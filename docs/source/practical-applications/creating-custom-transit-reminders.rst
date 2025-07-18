Creating Custom Transit Reminders Using the /records Endpoint
=============================================================

This guide explains how to use the Smartblitzmerker Open Data API to create personalized public transport reminders. We'll demonstrate how to fetch upcoming departures from a specific station using query parameters like ``where``, ``refine``, and ``order_by``, and automate alerts using tools such as ``cron`` or ``Node-RED``.

Overview
--------

The ``/records`` endpoint allows you to query real-time or scheduled transport data from SNCF datasets. By filtering for a specific station and ordering results by time, you can identify the next available departure and trigger a notification system.

Endpoint Reference
------------------

**GET** ``/datasets/{dataset_id}/records``

Required:
- ``dataset_id`` (e.g., `bus-timetables`, `train-departures`)

Common Parameters:
- ``where``: ODSQL condition to filter records (e.g., ``where=stop_name="Gare de Lyon"``)
- ``refine``: Faceted filter (e.g., ``refine=line_id:72``)
- ``order_by``: Sorting key, such as ``order_by=departure_time asc``
- ``limit``: Number of results to return
- ``timezone``: Adjust time to your local zone (e.g., ``timezone=Europe/Paris``)

Example Query
-------------

To fetch the next departures from *Gare de Lyon*, ordered by time:

.. code-block:: bash

   curl -G "https://ressources.data.sncf.com/api/explore/v2.1/catalog/datasets/train-departures/records" \
     --data-urlencode 'where=stop_name="Gare de Lyon"' \
     --data-urlencode 'order_by=departure_time asc' \
     --data-urlencode 'limit=5' \
     --data-urlencode 'timezone=Europe/Paris'

This request returns the next 5 scheduled departures from Gare de Lyon.

Automating Alerts
-----------------

You can automate this process to run periodically and trigger a notification (e.g., a desktop pop-up, SMS, or smart speaker output).

**Option 1: Using `cron` + `curl` + `notify-send` (Linux)**

1. Create a shell script (`check_departure.sh`):

.. code-block:: bash

   #!/bin/bash
   response=$(curl -sG "https://ressources.data.sncf.com/api/explore/v2.1/catalog/datasets/train-departures/records" \
     --data-urlencode 'where=stop_name="Gare de Lyon"' \
     --data-urlencode 'order_by=departure_time asc' \
     --data-urlencode 'limit=1' \
     --data-urlencode 'timezone=Europe/Paris')

   departure=$(echo "$response" | jq -r '.results[0].departure_time')
   notify-send "Next Departure from Gare de Lyon" "$departure"

2. Add to crontab:

.. code-block:: bash

   */5 * * * * /path/to/check_departure.sh

This will check every 5 minutes and display a desktop notification.

**Option 2: Using Node-RED**

1. Use an HTTP Request node to query the API.
2. Use a JSON node to parse the response.
3. Add a Function node to extract and format the departure time.
4. Connect to a notification output (e.g., email, Telegram bot, mobile push).

This setup offers a visual drag-and-drop approach to automate and route transit alerts.

Advanced: Filter by Upcoming Departures (Next 10 Minutes)
---------------------------------------------------------

To only fetch departures within the next 10 minutes, you can use a dynamic `where` clause with datetime comparison (server-side datetime functions are limited in the API, so apply this filtering locally if needed).

1. Fetch the earliest upcoming departure from a given stop.
2. Compare the result timestamp against the current time +10 minutes.
3. Trigger the alert only if within the time window.

.. code-block:: python

   import requests
   from datetime import datetime, timedelta
   import pytz

   now = datetime.now(pytz.timezone("Europe/Paris"))
   future = now + timedelta(minutes=10)

   r = requests.get(
       "https://ressources.data.sncf.com/api/explore/v2.1/catalog/datasets/train-departures/records",
       params={
           "where": 'stop_name="Gare de Lyon"',
           "order_by": "departure_time asc",
           "limit": 1,
           "timezone": "Europe/Paris"
       }
   )
   result = r.json()["results"][0]
   departure_time = datetime.fromisoformat(result["departure_time"])

   if now < departure_time < future:
       print(f"Upcoming train at {departure_time.strftime('%H:%M')}")
   else:
       print("No departure in next 10 minutes")

Conclusion
----------

The Smartblitzmerker API enables powerful transit tracking features with just a few well-placed queries. Combined with automation tools, this makes it easy to create a custom commuter assistant tailored to your schedule.
