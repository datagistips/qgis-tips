# See on Street View

This project enables you to activate a "See on Street View" action on a layer.

Just click on the object where you want to look, and here you go !

Just download the complete repository and open `see-on-street-view.qgs`

## Where the pegman will look
If the layer is polygonal, the pegman will look at the centroid of the object.

It's achieved by :

	lon2 = [%x(centroid(geometry($currentfeature)))%]
	lat2 = [%y(centroid(geometry($currentfeature)))%]

## Bearings with Google
The python code includes angle transformation function to get correct google bearings to use in Street View access.

The way Google formulates bearings is quite special

	def get_google_bearing(ang) :
		    ang2 = ang - 90 # + 90° # 0° sur Google correspond à 90°
		    ang3 = -ang2
		    delta = abs((180 - abs(ang3))) # différence par rapport au Sud
		    
		    if ang3 > 180  :
		        ang4 = -180 + delta
		    elif ang3 < -180 :
		        ang4 = 180 - delta,
		    else :
		        ang4 = ang3
		    
		    return(ang4)

To put all the code, go on `layer > Action > add Python action`, then paste the following code :

	from qgis.gui import QgsMapToolEmitPoint
	from qgis.utils import iface
	import webbrowser, math
	
	def transform_crs(lon1, lat1, epsg1=3857, epsg2=4326):
	    geom = QgsGeometry(QgsPoint(lon1, lat1))
	    sourceCrs = QgsCoordinateReferenceSystem(epsg1)
	    destCrs = QgsCoordinateReferenceSystem(epsg2)
	    tr = QgsCoordinateTransform(sourceCrs, destCrs, QgsProject.instance())
	    geom2 = QgsGeometry(geom)
	    geom2.transform(tr)
	    lon = geom2.asPoint().x()
	    lat = geom2.asPoint().y()
	    return((lon, lat))
	
	def get_google_bearing(ang) :
	    ang2 = ang - 90 # + 90° à cause de google. 0° sur Google correspond à 90°
	    ang3 = -ang2
	    delta = abs((180 - abs(ang3))) # différence par rapport au Sud
	    
	    if ang3 > 180  :
	        ang4 = -180 + delta
	    elif ang3 < -180 :
	        ang4 = 180 - delta,
	    else :
	        ang4 = ang3
	    
	    return(ang4)
	
	def display_point( pointTool ): 
	    print("Source coordinates")
	    lon1 = pointTool[0]
	    lat1 = pointTool[1]
	    lon1, lat1 = transform_crs(lon1, lat1)    
	    print(lon1)
	    print(lat1)
	    
	    print("Target coordinates")
	    lon2 = [%x(centroid(geometry($currentfeature)))%]
	    lat2 = [%y(centroid(geometry($currentfeature)))%]
	    #lon2, lat2 = transform_crs(lon2, lat2)
	    print(lon2)
	    print(lat2)
	
	    # Calculate angle
	    dx = lon2 - lon1
	    dy = lat2 - lat1
	    ang = math.atan2(dy, dx)
	    ang = (ang / math.pi) * 180
	    print("ang : ", ang)
	
	    # google bearing
	    ang_google = get_google_bearing(ang)
	    print("ang_google : ", ang_google)
	    
	    # google street view url
	    mode = 12
	    url = ("https://www.google.com/maps?layer=c&cbll=%f,%f&cbp=%f,%f,0,0,0")%(lat1, lon1, mode, ang_google)
	    # url='https://www.google.fr/%d&%d'%(1, 2)
	    webbrowser.open(url)  # Go to example.com
	
	    
	# a reference to our map canvas 
	canvas = iface.mapCanvas() 
	
	# this QGIS tool emits as QgsPoint after each click on the map canvas
	pointTool = QgsMapToolEmitPoint(canvas)
	
	pointTool.canvasClicked.connect(display_point)
	
	canvas.setMapTool( pointTool )
	
	    
	# a reference to our map canvas 
	canvas = iface.mapCanvas() 
	
	# this QGIS tool emits as QgsPoint after each click on the map canvas
	pointTool = QgsMapToolEmitPoint(canvas)
	
	pointTool.canvasClicked.connect(display_point)
	
	canvas.setMapTool( pointTool )


# Coordinates transformation between two EPSGs
You can notice I added a coordinates transformation helper function which, by default, transforms from `EPSG:3857` to `EPSG:4326`

	def transform_crs(lon1, lat1, epsg1=3857, epsg2=4326):
	    geom = QgsGeometry(QgsPoint(lon1, lat1))
	    sourceCrs = QgsCoordinateReferenceSystem(epsg1)
	    destCrs = QgsCoordinateReferenceSystem(epsg2)
	    tr = QgsCoordinateTransform(sourceCrs, destCrs, QgsProject.instance())
	    geom2 = QgsGeometry(geom)
	    geom2.transform(tr)
	    lon = geom2.asPoint().x()
	    lat = geom2.asPoint().y()
	    return((lon, lat))

	lon1, lat1 = transform_crs(lon1, lat1)    