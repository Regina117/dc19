# External Web Map Tile Server {: #data_external_wmts }

GeoServer has the ability to proxy a remote Web Map Tile Service (WMTS). This process is sometimes known as **Cascading WMTS**, even if the incoming requests follow the WMS protocol and the backing service follows the WMTS one; the WMTS cascading functionality is more like a "protocol translator", where the different handled data (capabilities documents, images) are translated by the "WMTS cascading" logic.

Loading a remote WMTS is useful for many reasons. If you don't manage or have access to the remote WMTS, you can now manage its output as if it were local. Even if you don't have any control on the remote WMTS, you can use GeoServer features to treat its output (watermarking, decoration, printing, etc).

To access a remote WMTS, it is necessary to load it as a store in GeoServer. GeoServer must be able to access the capabilities document of the remote WMTS for the store to be successfully loaded.

## Adding an external WMTS

To connect to an external WMTS, it is necessary to load it as a new store. To start, in the [Web administration interface](../../webadmin/index.md), navigate to **Stores --> Add a new store --> WMTS**. The option is listed under **Other Data Sources**.

![](images/wmtsaddnew.png)
*Adding an external WMTS as a store*

![](images/wmtsconfigure.png)
*Configuring a new external WTMS store*

  -------------------------------- ------------------------------------------------------------------------------------------------------------------------------
  **Option**                       **Description**

  **Workspace**                    Name of the workspace to contain the store. This will also be the prefix of all of the layer names published from the store.

  **Data Source Name**             Name of the store as known to GeoServer.

  **Description**                  Description of the store.

  **Enabled**                      Enables the store. If disabled, no data from the remote WMTS will be served.

  **Capabilities URL**             The full URL to access the capabilities document of the remote WMTS.

  **User Name**                    If the WMTS requires authentication, the user name to connect as.

  **Password**                     If the WMTS requires authentication, the password to connect with.

  **HTTP header name**             If the WMTS requires a custom HTTP header, the header name.

  **HTTP header value**            If the WMTS requires a custom HTTP header, the header value.

  **Max concurrent connections**   The maximum number of persistent connections to keep for this WMTS.
  -------------------------------- ------------------------------------------------------------------------------------------------------------------------------

When finished, click **Save**.

## Configuring external WMTS layers

When properly loaded, all layers served by the external WMTS will be available to GeoServer. Before they can be served, however, they will need to be individually configured (published) as new layers. See the section on [Layers](../webadmin/layers.md) for how to add and edit new layers. Once published, these layers will show up in the [Layer Preview](../webadmin/layerpreview.md) and as part of the WMS capabilities document. If the WMTS layer has additional dimensions (e.g. time), related info will be reported on the WMS capabilities as well.

## Features

Connecting a remote WMTS allows for the following features:

-   **Dynamic reprojection**. While the default projection for a layer is cascaded, it is possible to pass the SRS parameter through to the remote WMS. Should that SRS not be valid on the remote server, GeoServer will dynamically reproject the tiles sent to it from the remote WMTS.
-   Full **REST Configuration**. See the [REST](../../rest/index.md) section for more information about the GeoServer REST interface.

## Limitations

Layers served through an external WMTS have some, but not all of the functionality of a local layer.

-   Layers cannot be styled with SLD.
-   Alternate (local) styles cannot be used.
-   GetFeatureInfo requests aren't supported.
-   GetLegendGraphic requests aren't supported.
-   Image format cannot be specified. GeoServer will attempt to request PNG images, and if that fails will use the remote server's default image format.

## Images output discrepancies in a cascaded WMTS Layer

WMTS it is a service that serves tiles and they have been generated for a concrete resolution/scale denominator. Asking a WMTS cascaded layer to generate WMS GetMap images or other WMTS tiles, with other scale denominators, will require image re-sampling:

-   If the image is stretched (scaled out) and the scale difference is notable, the borders, lines, and labels that appear in it could be blurred.
-   On the other hand if shrunk, the same object and shape could appear smaller than the original size and will be similarly appear blurred.

![](images/cascaded_wmts.png)
*This figure compares the resulting image from a WMTS to a cascaded layer which has been slightly stretched or scaled out**Left image shows a original wmts layer at its defined zoom level 4 which scale denominator is about 4M*
*Right image shows a cascaded wmts layer as wms layer with at different scale denominator (the closest to its homologous cascaded layer) which is about 5M*