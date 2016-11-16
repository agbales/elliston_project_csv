# elliston_project_csv

Setup

1) Place the new CSV file in the ‘data’ project folder.

2) In the Elliston_CSV project, open /js/index.js . Be sure that var csvfile inside the document ready function is pointing towards the current .csv from the database.

3) Use MAMP (or local server) to run the Elliston_CSV project.


Usage

Once you open the project, you’ll have a few options:

Grab the code for select entries.

Clicking “Display Listings” will list the code for each listing. This can be cut/pasted into the code body of the WP post.

Retrieve a JavaScript object that contains the entire DB.

In the browser’s console, you may save the first logged variable ’ellistonData’ (see instructions below), as an object. It will have the following structure:

PlaylistID
	- - > Listing number
		- - > Authors (array)
		- - > Listing (HTML)
		- - > playlistHTML (HTML for WP player)
		- - > Tracks (Array of objects)  - - > mp3 (link) 
								   - - > trackNum

Retrieve a JSON object (required to update the WP posts).

In the browser’s console, you may save the first logged variable ’ellistonData’ (see instructions below), as an object. It will have the following structure:

In the browser’s console, you may save the second logged variable ‘el’ (see instructions below), which contains information to create new WP posts. It’s an array with objects that follows this structure:

Array
	- - > Object
		- - > Author
		- - > authorInfoPlaceholder
		- - > drcLink
		- - > playlist (HTML for WP plugin)


Saving variables from the console:

1) Right click the object and save as a global variable.
2) Paste the following function in the console and press ‘enter’:

(function(console){

console.save = function(data, filename){

    if(!data) {
        console.error('Console.save: No data')
        return;
    }

    if(!filename) filename = 'console.json'

    if(typeof data === "object"){
        data = JSON.stringify(data, undefined, 4)
    }

    var blob = new Blob([data], {type: 'text/json'}),
        e    = document.createEvent('MouseEvents'),
        a    = document.createElement('a')

    a.download = filename
    a.href = window.URL.createObjectURL(blob)
    a.dataset.downloadurl =  ['text/json', a.download, a.href].join(':')
    e.initMouseEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null)
    a.dispatchEvent(e)
 }
})(console)

3) To download, pass the name of the variable (ex: temp1) and the desired file name (ex: ‘elliston_JSON’) as arguments to console.save in the console:

	console.save(temp1, ‘elliston_JSON’)


Insert WP posts

Once you’ve saved the JSON object, you can now update the WP site. Go to the posts section of the dashboard in WP. Then function.php modify the file located at:

	../wp-content/themes/CURRENT-THEME/functions.php

At the bottom of the file, add the code:

if ( true !== DOING_CRON && true !== DOING_AJAX ) { 

	$json = “PASTE (or direct to file for) JSON OBJECT SAVED FROM CONSOLE”;

 	$arr = json_decode( $json, true );

 	for ($item = 0; $item < count($arr); $item++) {
 		$author = $arr[$item]['author'];
		$drcLink = $arr[$item]['drcLink'];
 		$linkBreak = "<br><br>";
		$postHTML = $arr[$item]['playlist'] . 
					$linkBreak . 
					"Find the DRC listing here: <a href='" . $drcLink . "'> " . $drcLink . "</a>";

	    $id = wp_insert_post(array(
 		 'post_title'    => $author,
 		 'post_content'  => $postHTML,
 		 'post_author'   => 'admin',
 		 'post_type'     => 'post',
 		 'post_status'   => 'publish',
 		));
 	}
 }


Save & refresh the dashboard. **IMPORTANT** It’s possible you’ll get an error message. (“Though the bag may not inflate, oxygen is flowing to the mask”). Regardless, DO NOT refresh again, as EVERY refresh of the page will add the entire array of posts. Before you return to the page, you’ll need to comment out or remove the functions.php insert code we just added, re-save, and then refresh the dashboard. The posts should now be populated!

