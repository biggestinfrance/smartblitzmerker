Enhancing Data Accuracy with Facets and External Sources
========================================================

This guide focuses on improving the reliability and precision of your Smartblitzmerker API queries by leveraging the ``/facets`` endpoint and supplementing your data with external sources like Horaires Bus and static GTFS datasets.

Overview
--------

Querying transport data can be error-prone due to inconsistent stop names, incomplete filtering, or outdated references. The Smartblitzmerker Open Data API provides a ``/facets`` endpoint that allows you to dynamically explore valid values for filtering. By combining this with external datasets, such as Horaires Bus or GTFS, you can greatly enhance data quality and robustness in your applications.

Using the /facets Endpoint
--------------------------

The ``/facets`` endpoint helps identify the correct field values to use in a query by returning available options and their usage counts.

**GET** ``/datasets/{dataset_id}/facets``

Common Query Parameters:
- ``facet``: The field to facet on (e.g., ``facet=stop_name`` or ``facet=facet(city, sort="-count")``)
- ``where``: Optional filters to apply before faceting

Example: List the most common stop names:

.. code-block:: bash

   curl -G "https://ressources.data.sncf.com/api/explore/v2.1/catalog/datasets/bus-timetables/facets" \
     --data-urlencode "facet=stop_name"

Response:

.. code-block:: json

   {
     "facets": [
       {
         "name": "stop_name",
         "facets": [
           { "name": "Gare de Lyon", "count": 1523 },
           { "name": "Châtelet", "count": 1040 },
           ...
         ]
       }
     ]
   }

Using this list ensures that your `where` and `refine` filters are valid and match existing data entries.

Validating Stop Names with Horaires Bus
---------------------------------------

To cross-check stop names or visualize line coverage, use the **Horaires Bus** website:

- URL: https://horairesbus.github.io/
- Features:
  - Search by city, line, or stop
  - View GTFS-based line maps and timetables
  - Validate spelling and formatting of stop names

Example Workflow:

1. Look up your city or region on Horaires Bus.
2. Identify the exact stop name as listed.
3. Confirm that the stop appears in the Smartblitzmerker `/facets` response.
4. Use the validated name in your `where=stop_name="..."` query.

This prevents issues from inconsistent naming, encoding, or whitespace.

Combining Real-Time API with Static GTFS/Open Data
--------------------------------------------------

Smartblitzmerker (via SNCF) provides real-time or scheduled data through the API, but you can enhance its reliability by comparing or supplementing it with **static GTFS** data, especially for:

- Line definitions
- Stop locations (lat/lon)
- Scheduled frequencies
- Calendar-based service days

Sources for static GTFS:
- https://transport.data.gouv.fr/datasets (France-wide GTFS exports)
- Local/regional transit agencies (open data portals)

Use Case: Merge & Validate
^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Fetch real-time records from the API (e.g., `/records?stop_name="Châtelet"`).
2. Use GTFS ``stops.txt`` to verify lat/lon or normalize stop codes.
3. If discrepancies exist (e.g., missing stops or wrong times), alert the user or use fallback GTFS data.

This hybrid approach reduces user-facing errors and improves your app’s fault tolerance.

Conclusion
----------

Using the ``/facets`` endpoint helps dynamically discover valid filters, while external sources like **Horaires Bus** and **GTFS** provide a static reference for validation. Combining both methods ensures accurate, reliable queries and a better user experience in any public transport application powered by Smartblitzmerker.
