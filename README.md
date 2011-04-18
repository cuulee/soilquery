# soilquery readme
2011-04-17
Bill Morris

# Project Components
## Input Data
Soil chemistry and biophysical data provided by ground surveys or by any publicly-available soil data source. The initial format is 1-meter resolution .tif raster, of which there are about 80 layers representing everything from nitrogen content to soil texture at various horizons.
## GUI
A web map interface based on [probably google] aerial imagery, with a really basic toolkit. A user should be able to select a drawing tool and a desired soil characteristics layer, sketch a "management boundary" (for instance a paddock she wants to plant with alfalfa for the coming year), and hit a "calculate" button. The result should then be a report, telling the user how big the paddock is, along with mean, minimum and maximum values for the selected raster within the geometry of the polygon. Ideally, the user should be able to come back to the page in the future and compare past years' management boundaries and results through the toolkit.

# Workflow
## Drawing a polygon
User draws vector polygon on Google Maps API v3 imagery:
    <head> 
    <meta name="viewport" content="width=device-width; initial-scale=1.0; maximum-scale=1.0; user-scalable=0;" /> 
    <meta name="apple-mobile-web-app-capable" content="yes" /> 
    <title>Modify Feature</title> 
    <link rel="stylesheet" href="../theme/default/style.css" type="text/css" /> 
    <link rel="stylesheet" href="style.css" type="text/css" /> 
    <style type="text/css"> 
        #controls {
            width: 512px;
        }
        #controlToggle {
            padding-left: 1em;
        }
        #controlToggle li {
            list-style: none;
        }
    </style> 
    <script src="../lib/Firebug/firebug.js"></script> 
    <script src="../OpenLayers.js"></script> 
    <script type="text/javascript"> 
        var map, vectors, controls;
        function init(){
            map = new OpenLayers.Map('map');
            var wms = new OpenLayers.Layer.WMS( "OpenLayers WMS", 
                "http://vmap0.tiles.osgeo.org/wms/vmap0?", {layers: 'basic'}); 
            OpenLayers.Feature.Vector.style['default']['strokeWidth'] = '2';
 
            // allow testing of specific renderers via "?renderer=Canvas", etc
            var renderer = OpenLayers.Util.getParameters(window.location.href).renderer;
            renderer = (renderer) ? [renderer] : OpenLayers.Layer.Vector.prototype.renderers;
 
            vectors = new OpenLayers.Layer.Vector("Vector Layer", {
                renderers: renderer
            });
 
            map.addLayers([wms, vectors]);
            map.addControl(new OpenLayers.Control.LayerSwitcher());
            map.addControl(new OpenLayers.Control.MousePosition());
            
            function report(event) {
                OpenLayers.Console.log(event.type, event.feature ? event.feature.id : event.components);
            }
            vectors.events.on({
                "beforefeaturemodified": report,
                "featuremodified": report,
                "afterfeaturemodified": report,
                "vertexmodified": report,
                "sketchmodified": report,
                "sketchstarted": report,
                "sketchcomplete": report
            });
            controls = {
                point: new OpenLayers.Control.DrawFeature(vectors,
                            OpenLayers.Handler.Point),
                line: new OpenLayers.Control.DrawFeature(vectors,
                            OpenLayers.Handler.Path),
                polygon: new OpenLayers.Control.DrawFeature(vectors,
                            OpenLayers.Handler.Polygon),
                regular: new OpenLayers.Control.DrawFeature(vectors,
                            OpenLayers.Handler.RegularPolygon,
                            {handlerOptions: {sides: 5}}),
                modify: new OpenLayers.Control.ModifyFeature(vectors)
            };
            
            for(var key in controls) {
                map.addControl(controls[key]);
            }
            
            map.setCenter(new OpenLayers.LonLat(0, 0), 3);
            document.getElementById('noneToggle').checked = true;
        }
        
        function update() {
            // reset modification mode
            controls.modify.mode = OpenLayers.Control.ModifyFeature.RESHAPE;
            var rotate = document.getElementById("rotate").checked;
            if(rotate) {
                controls.modify.mode |= OpenLayers.Control.ModifyFeature.ROTATE;
            }
            var resize = document.getElementById("resize").checked;
            if(resize) {
                controls.modify.mode |= OpenLayers.Control.ModifyFeature.RESIZE;
                var keepAspectRatio = document.getElementById("keepAspectRatio").checked;
                if (keepAspectRatio) {
                    controls.modify.mode &= ~OpenLayers.Control.ModifyFeature.RESHAPE;
                }
            }
            var drag = document.getElementById("drag").checked;
            if(drag) {
                controls.modify.mode |= OpenLayers.Control.ModifyFeature.DRAG;
            }
            if (rotate || drag) {
                controls.modify.mode &= ~OpenLayers.Control.ModifyFeature.RESHAPE;
            }
            var sides = parseInt(document.getElementById("sides").value);
            sides = Math.max(3, isNaN(sides) ? 0 : sides);
            controls.regular.handler.sides = sides;
            var irregular =  document.getElementById("irregular").checked;
            controls.regular.handler.irregular = irregular;
        }
 
        function toggleControl(element) {
            for(key in controls) {
                var control = controls[key];
                if(element.value == key && element.checked) {
                    control.activate();
                } else {
                    control.deactivate();
                }
            }
        }
        
    </script> 
  </head> 
  <body onload="init()"> 
    <h1 id="title">OpenLayers Modify Feature Example</h1> 
    <div id="tags"> 
        vertices, digitizing, draw, drawing
    </div> 
    <div id="shortdesc">A demonstration of the ModifyFeature control for editing vector features.</div> 
    <div id="map" class="smallmap"></div> 
    <div id="controls"> 
        <ul id="controlToggle"> 
            <li> 
                <input type="radio" name="type" value="none" id="noneToggle"
                       onclick="toggleControl(this);" checked="checked" /> 
                <label for="noneToggle">navigate</label> 
            </li> 
            <li> 
                <input type="radio" name="type" value="point" id="pointToggle" onclick="toggleControl(this);" /> 
                <label for="pointToggle">draw point</label> 
            </li> 
            <li> 
                <input type="radio" name="type" value="line" id="lineToggle" onclick="toggleControl(this);" /> 
                <label for="lineToggle">draw line</label> 
            </li> 
            <li> 
                <input type="radio" name="type" value="polygon" id="polygonToggle" onclick="toggleControl(this);" /> 
                <label for="polygonToggle">draw polygon</label> 
            </li> 
            <li> 
                <input type="radio" name="type" value="regular" id="regularToggle" onclick="toggleControl(this);" /> 
                <label for="regularToggle">draw regular polygon</label> 
                <label for="sides"> - sides</label> 
                <input id="sides" type="text" size="2" maxlength="2"
                       name="sides" value="5" onchange="update()" /> 
                <ul> 
                    <li> 
                        <input id="irregular" type="checkbox"
                               name="irregular" onchange="update()" /> 
                        <label for="irregular">irregular</label> 
                    </li> 
                </ul> 
            </li> 
            <li> 
                <input type="radio" name="type" value="modify" id="modifyToggle"
                       onclick="toggleControl(this);" /> 
                <label for="modifyToggle">modify feature</label> 
                <ul> 
                    <li> 
                        <input id="rotate" type="checkbox"
                               name="rotate" onchange="update()" /> 
                        <label for="rotate">allow rotation</label> 
                    </li> 
                    <li> 
                        <input id="resize" type="checkbox"
                               name="resize" onchange="update()" /> 
                        <label for="resize">allow resizing</label> 
                        (<input id="keepAspectRatio" type="checkbox"
                               name="keepAspectRatio" onchange="update()" checked="checked" /> 
                        <label for="keepAspectRatio">keep aspect ratio</label>)
                    </li> 
                    <li> 
                        <input id="drag" type="checkbox"
                               name="drag" onchange="update()" /> 
                        <label for="drag">allow dragging</label> 
                    </li> 
                </ul> 
            </li> 
        </ul> 
    </div> 
  </body> 
## Passing the geometry
Polygon geometry is passed to a database, also stored there for future graphic retrieval
## Agoodle query
Polygon is used to query soil raster data stored in the same database via agoodle (https://github.com/brentp/agoodle)
## Query result storage
Calculation results are stored in the database associated with the polygon
## To-User onscreen query output
Results are written to a neat-looking report onscreen for the user, perhaps with a bar chart graphic and/or an option to download an .xls.