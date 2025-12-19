# Website contributing guide
The website is written in markdown format, created through sphinx. These markdown files can be edited/created/deleted to update/create/remove pages.

In order to contribute to the website, a series of steps should be followed:

<ol>
  <li>Download the code from the website repository</li>
  <li>Create a new branch based off the Dev branch.</li>
  <li>Make the desired changes to the markdown files located within the source directory.</li>
  <li>Preview your changes through: 
    <ul>
      <li>your IDE of choice (look it up for your ide. 
<a href="https://code.visualstudio.com/docs/languages/markdown#_markdown-preview">Visual Studio Code Example</a>)</li>
      <li>building the website through sphinx and then viewing the html</li>
      <li>any other markdown viewing program</li>
    </ul>
  </li>
  <li>Upload your changes to a separate branch</li>
  <li>Create a pull request to the Dev branch</li>
  <li>(Optional) Create a pull request from the Dev branch into the Main branch whenever you are ready for the website to be updated</li>
</ol> 

## Adding images to your page
Every image that is contained within the website should be stored within the `_static` folder located within source.
In order to then use that image you must reference it's location within the markdown page.

As an example:
```
<div align="center">
	<img align="center" src="../_static/img/Iilluminator.jpg" width="500">
</div>
```

Which results in a centered image, of width 500 like so:
<div align="center">
	<img align="center" src="../_static/img/illuminator.jpg" width="500">
</div>
 

## Building and previewing the website through sphinx
Building and previewing the website before it is uploaded on github is highly recommended. In order to create this preview a series of steps must be followed. Example over command line interface:

<ol>
  <li>Ensure you are in the correct directory with your terminal (your directory should see README.md and source)</li>
  <li>Prepare your environment:
    <ol>
      <li>python3 -m venv .venv</li>
      <li>Activate your work environment:
        <ul>
            <li>MacOS & Linux: source .venv/bin/activate</li>
            <li>Windows: .venv\Scripts\activate </li>
        </ul>
        </li>
      <li>pip install -r source/requirements.txt</li>
      <li>pip install illuminator --no-deps</li>
    </ol>
  </li>
  <li>Make your changes (if you added a new page make sure to add it to index.rst so its visible in the side-bar)</li>
  <li>Build using sphinx:
    <ul>
        <li>sphinx-build source docs (NOTE: the name docs can be changed if you wish)</li>
        <li>sphinx-build source docs --fail-on-warning (optional flag, sometimes it works sometimes it does not. The warnings do not break the website build)</li> 
    </ul>
  </li>
  <li>Open any html document located within your new folder using your web browser to preview your website</li>
  <li>[Important] After you are done and want to push your changes to github you must ensure that you either delete the folder which was created when building this website, or add it to .gitignore if you plan on reusing it (see .gitignore file for examples)</li>
</ol> 
