# Beach-Erosion-Analyzer
refer https://github.com/kvos/CoastSat for detailed information about CoastSat


1. Installation
1.1 Create an environment with Anaconda
To run the toolbox you first need to install the required Python packages in an environment. To do this we will use Anaconda, which can be downloaded freely here.

Once you have it installed on your PC, open the Anaconda prompt (in Mac and Linux, open a terminal window) and use the cd command (change directory) to go the folder where you have downloaded this repository.

Create a new environment named coastsat with all the required packages:

conda env create -f environment.yml -n coastsat
All the required packages have now been installed in an environment called coastsat. Now, activate the new environment:

conda activate coastsat
To confirm that you have successfully activated CoastSat, your terminal command line prompt should now start with (coastsat).

1.2 Activate Google Earth Engine Python API
irst, you need to request access to Google Earth Engine at https://signup.earthengine.google.com/. It takes about 1 day for Google to approve requests.

Once your request has been approved, with the coastsat environment activated, run the following command on the Anaconda Prompt to link your environment to the GEE server:
Run me first
First of all, run the following cell to initialize the API. The output will contain instructions on how to grant this notebook access to Earth Engine using your account.
A web browser will open, login with a gmail account and accept the terms and conditions. Then copy the authorization code into the input.

2. Usage
An example of how to run the software in a Jupyter Notebook is provided in the repository (example_jupyter.ipynb). To run this, first activate your coastsat environment with conda activate coastsat (if not already active), and then type:

jupyter notebook
A web browser window will open. Point to the directory where you downloaded this repository and click on example_jupyter.ipynb.

The following sections guide the reader through the different functionalities of CoastSat with an example at Narrabeen-Collaroy beach (Australia). If you prefer to use Spyder, PyCharm or other integrated development environments (IDEs), a Python script named example.py is also included in the repository.

If using example.py on Spyder, make sure that the Graphics Backend is set to Automatic and not Inline (as this mode doesn't allow to interact with the figures). To change this setting go under Preferences>IPython console>Graphics.

A Jupyter Notebook combines formatted text and code. To run the code, place your cursor inside one of the code sections and click on the run cell button (or press Shift + Enter) and progress forward.

2.1 Retrieval of the satellite images
To retrieve from the GEE server the available satellite images cropped around the user-defined region of coastline for the particular time period of interest, the following variables are required:

polygon: the coordinates of the region of interest (longitude/latitude pairs in WGS84)
dates: dates over which the images will be retrieved (e.g., dates = ['2019-12-01', '2021-01-01'])
sat_list: satellite missions to consider (e.g., sat_list = ['L5', 'L7', 'L8', 'S2'] for Landsat 5, 7, 8 and Sentinel-2 collections)
sitename: name of the site (this is the name of the subfolder where the images and other accompanying files will be stored)
filepath: filepath to the directory where the data will be stored
The call metadata = SDS_download.retrieve_images(inputs) will launch the retrieval of the images and store them as .TIF files (under /filepath/sitename). The metadata contains the exact time of acquisition (in UTC time) of each image, its projection and its geometric accuracy. If the images have already been downloaded previously and the user only wants to run the shoreline detection, the metadata can be loaded directly by running metadata = SDS_download.get_metadata(inputs).

2.2 Shoreline detection
To map the shorelines, the following user-defined settings are needed:

cloud_thresh: threshold on maximum cloud cover that is acceptable on the images (value between 0 and 1 - this may require some initial experimentation).
output_epsg: epsg code defining the spatial reference system of the shoreline coordinates. It has to be a cartesian coordinate system (i.e. projected) and not a geographical coordinate system (in latitude and longitude angles). See http://spatialreference.org/ to find the EPSG number corresponding to your local coordinate system. If unsure, use 3857 which is the web-mercator.
check_detection: if set to True the user can quality control each shoreline detection interactively (recommended when mapping shorelines for the first time) and accept/reject each shoreline.
adjust_detection: in case users wants more control over the detected shorelines, they can set this parameter to True, then they will be able to manually adjust the threshold used to map the shoreline on each image.
save_figure: if set to True a figure of each mapped shoreline is saved under /filepath/sitename/jpg_files/detection, even if the two previous parameters are set to False. Note that this may slow down the process.
There are additional parameters (min_beach_size, buffer_size, min_length_sl, cloud_mask_issue and sand_color) that can be tuned to optimise the shoreline detection (for Advanced users only). For the moment leave these parameters set to their default values, we will see later how they can be modified.

When check_detection is set to True, a figure like the one below appears and asks the user to manually accept/reject each detection by pressing on the keyboard the right arrow (⇨) to keep the shoreline or left arrow (⇦) to skip the mapped shoreline. The user can break the loop at any time by pressing escape (nothing will be saved though).

When adjust_detection is set to True, a figure like the one below appears and the user can adjust the position of the shoreline by clicking on the histogram of MNDWI pixel intensities. Once the threshold has been adjusted, press Enter and then accept/reject the image with the keyboard arrows.

Once all the shorelines have been mapped, the output is available in two different formats (saved under /filepath/data/sitename):

sitename_output.pkl: contains a list with the shoreline coordinates, the exact timestamp at which the image was captured (UTC time), the geometric accuracy and the cloud cover of each individual image. This list can be manipulated with Python, a snippet of code to plot the results is provided in the example script.
sitename_output.geojson: this output can be visualised in a GIS software (e.g., QGIS, ArcGIS).
The figure below shows how the satellite-derived shorelines can be opened in a GIS software (QGIS) using the .geojson output. Note that the coordinates in the .geojson file are in the spatial reference system defined by the output_epsg.

Reference shoreline
Before running the batch shoreline detection, there is the option to manually digitize a reference shoreline on one cloud-free image. This reference shoreline helps to reject outliers and false detections when mapping shorelines as it only considers as valid shorelines the points that are within a defined distance from this reference shoreline.

The user can manually digitize one or several reference shorelines on one of the images by calling:

settings['reference_shoreline'] = SDS_preprocess.get_reference_sl_manual(metadata, settings)
settings['max_dist_ref'] = 100 # max distance (in meters) allowed from the reference shoreline
This function allows the user to click points along the shoreline on cloud-free satellite images, as shown in the animation below.

The maximum distance (in metres) allowed from the reference shoreline is defined by the parameter max_dist_ref. This parameter is set to a default value of 100 m. If you think that 100 m buffer from the reference shoreline will not capture the shoreline variability at your site, increase the value of this parameter. This may be the case for large nourishments or eroding/accreting coastlines.

Advanced shoreline detection parameters
As mentioned above, there are some additional parameters that can be modified to optimise the shoreline detection:

min_beach_area: minimum allowable object area (in metres^2) for the class 'sand'. During the image classification, some features (for example, building roofs) may be incorrectly labelled as sand. To correct this, all the objects classified as sand containing less than a certain number of connected pixels are removed from the sand class. The default value is 4500 m^2, which corresponds to 20 connected pixels of 15 m^2. If you are looking at a very small beach (<20 connected pixels on the images), try decreasing the value of this parameter.
buffer_size: radius (in metres) that defines the buffer around sandy pixels that is considered to calculate the sand/water threshold. The default value of buffer_size is 150 m. This parameter should be increased if you have a very wide (>150 m) surf zone or inter-tidal zone.
min_length_sl: minimum length (in metres) of shoreline perimeter to be valid. This can be used to discard small features that are detected but do not correspond to the actual shoreline. The default value is 200 m. If the shoreline that you are trying to map is shorter than 200 m, decrease the value of this parameter.
cloud_mask_issue: the cloud mask algorithm applied to Landsat images by USGS, namely CFMASK, does have difficulties sometimes with very bright features such as beaches or white-water in the ocean. This may result in pixels corresponding to a beach being identified as clouds and appear as masked pixels on your images. If this issue seems to be present in a large proportion of images from your local beach, you can switch this parameter to True and CoastSat will remove from the cloud mask the pixels that form very thin linear features, as often these are beaches and not clouds. Only activate this parameter if you observe this very specific cloud mask issue, otherwise leave to the default value of False.
sand_color: this parameter can take 3 values: default, dark or bright. Only change this parameter if you are seing that with the default the sand pixels are not being classified as sand (in orange). If your beach has dark sand (grey/black sand beaches), you can set this parameter to dark and the classifier will be able to pick up the dark sand. On the other hand, if your beach has white sand and the default classifier is not picking it up, switch this parameter to bright. At this stage this option is only available for Landsat images (soon for Sentinel-2 as well).
Re-training the classifier
CoastSat's shoreline mapping alogorithm uses an image classification scheme to label each pixel into 4 classes: sand, water, white-water and other land features. While this classifier has been trained using a wide range of different beaches, it may be that it does not perform very well at specific sites that it has never seen before. You can train a new classifier with site-specific training data in a few minutes by following the Jupyter notebook in re-train CoastSat classifier.

2.3 Shoreline change analysis
This section shows how to obtain time-series of shoreline change along shore-normal transects. Each transect is defined by two points, its origin and a second point that defines its length and orientation. The origin is always defined first and located landwards, the second point is located seawards. There are 3 options to define the coordinates of the transects:

Interactively draw shore-normal transects along the mapped shorelines:
transects = SDS_transects.draw_transects(output, settings)
Load the transect coordinates from a .geojson file:
transects = SDS_tools.transects_from_geojson(path_to_geojson_file)
Create the transects by manually providing the coordinates of two points:
transects = dict([])
transects['Transect 1'] = np.array([[342836, ,6269215], [343315, 6269071]])
transects['Transect 2'] = np.array([[342482, 6268466], [342958, 6268310]])
transects['Transect 3'] = np.array([[342185, 6267650], [342685, 6267641]])
Note: if you choose option 2 or 3, make sure that the points that you are providing are in the spatial reference system defined by settings['output_epsg'].

Once the shore-normal transects have been defined, the intersection between the 2D shorelines and the transects is computed with the following function:

settings['along_dist'] = 25
cross_distance = SDS_transects.compute_intersection(output, transects, settings)
The parameter along_dist defines the along-shore distance around the transect over which shoreline points are selected to compute the intersection. The default value is 25 m, which means that the intersection is computed as the median of the points located within 25 m of the transect (50 m alongshore-median). This helps to smooth out localised water levels in the swash zone.

