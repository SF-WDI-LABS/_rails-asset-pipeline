# Rails Asset Pipeline Exercises

### Asset Pipeline Poem

Start with the [asset pipeline poem lab](https://github.com/sf-wdi-34/asset-pipeline-poem).  Move on to the "Take it Further!" challenges below when you have these steps complete:

1. Put the lines of the file and title in the correct order.  
2. Show the character count (see screenshot in lab readme).  
3. Precompile so that all of your JavaScript and CSS assets are combined into one file each.

### Take it Further!

#### Displaying Images

 1. Add two images of your choice to your `app/assets/images` folder
 2. Add one image to your main site index view by calling the file name in an `image_tag` erb view helper.
 
  ```
    <%= image_tag 'rails.png' %>
  ```
 
 1. Check out [other asset tag helpers](http://api.rubyonrails.org/classes/ActionView/Helpers/AssetTagHelper.html), and add one more of any kind (could be another image) to your asset pipeline poem.

#### Using `asset_path`

1. Use jQuery to add a click event handler to the title of the poem.  Start by logging a sanity check message in your console. Test this.

1. Update the click event handler to change the image displayed on your index page.  Use the `asset_path` helper to access the correct path for the asset from inside your JavaScript code.
