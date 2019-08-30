**Import data in OMERO using the Command Line Interface (CLI)**
===============================================================

Description:
------------

This chapter will show how to import data for another user, using Command Line Interface (CLI).

The user importing the data needs to have some admin (or restricted-admin) privileges. More information about restricted privileges can be found at \ https://docs.openmicroscopy.org/latest/omero/sysadmins/restricted-admins.html

The import for another user will be done only as demo since the user is required to have specific privileges. We will use a user with login name importer1, who could be in real life e.g. a facility manager\ .

We will show:

-  How to import data using the CLI for myself and for others

-  How to import data using the CLI “in-place”, which means not copying the imported data into OMERO. Instead, OMERO will point to the original location of “in-place” imported files, thus preventing data duplication.

-  How to deal with imports of large amounts of data in CLI, using the --bulk option and helper csv and yml files which define what is to be imported and how.

**Resources:**
--------------

-  Documentation:

   -  https://docs.openmicroscopy.org/latest/omero/users/cli/installation.html

   -  `https://docs.openmicroscopy.org/omero/latest/users/cli/index.html <https://docs.openmicroscopy.org/omero/5.5.1/users/cli/index.html>`__

   -  `https://docs.openmicroscopy.org/omero/latest/users/cli/import-target.html <https://docs.openmicroscopy.org/omero/5.5.1/users/cli/import-target.html>`__

   -  `https://docs.openmicroscopy.org/omero/latest/sysadmins/in-place-import.html <https://docs.openmicroscopy.org/omero/5.5.1/sysadmins/in-place-import.html>`__

   -  `https://docs.openmicroscopy.org/omero/latest/users/cli/import-bulk.html <https://docs.openmicroscopy.org/omero/5.5.1/users/cli/import-bulk.html>`__

-  Data: example images from

   -  https://downloads.openmicroscopy.org/images/DV/will/FRAP/

-  Bash script for performing in-place imports:

   -  https://github.com/ome/training-scripts/blob/master/maintenance/scripts/in_place_import_as.sh

-  Example files for bulk import

   -  `https://github.com/ome/training-scripts/tree/latest/practical/other <https://github.com/ome/training-scripts/tree/v0.6.0/practical/other>`__

Setup:
------

**CLI Importer installation**

Client libraries from the OMERO.server have to be installed on the client to import images using CLI. The installation instructions can be
found at \ https://docs.openmicroscopy.org/latest/omero/users/cli/installation.html\ .

Note: When importing for another user using the CLI, the importer1 does not have to be a member of the target group.


**Step-by-step:**
-----------------

1.  Open a terminal and connect to the server as importer1 using ssh.

2.  The aim is to import an image from /OMERO/in-place-import/FRAP
       $ ls /OMERO/in-place-import/FRAP

3.  Go to $ cd /opt/omero/server/OMERO.server/bin

4.  The importer1 user logs in as themselves

    a. $ ./omero -u importer1 login

5.  Creates a Dataset import_for_myself

    b. $ DID=$(./omero obj new Dataset name=import_for_myself)

6.  Import the data in the newly created Dataset:

    c. $ ./omero import -d $DID /OMERO/in-place-import/FRAP/U20S-RCC1.10_R3D_FRAP.dv

7.  The importer1 user logs in as user-1:

    d. $ ./omero --sudo importer1 -u user-1 login

8.  Create a Dataset import_for_user_one as user-1:

    e. $ DID=$(./omero obj new Dataset name=import_for_user_one)

9.  Import the data in the newly created Dataset:

    f. $ ./omero import -d $DID /OMERO/in-place-import/FRAP/U20S-RCC1.10_R3D_FRAP.dv

10. Check that the images are successfully imported.

**In-place Import CLI** 
========================

Instead of being copied into OMERO’s managed repository, the image files
stay at their original place and are just linked into the repository.

It is only available for the CLI importer, using the argument --transfer=ln_s .

   .. image:: images/importcli1.png

**Advantages:**

-  All in-place import scenarios provide non-copying benefit. Data that is too large toxist in multiple places, or which is accessed too frequently in its original form to be renamed,remains where it was originally acquired.

**Limitations:**

-  Only available on the OMERO server system itself.

-  Do not edit or move the files after an in-place import. OMERO may no longer be able to access them if you do.

**Important:**

Someone wanting to perform an in-place import MUST have:

-  a regular OMERO account

-  an OS-account with access to bin/omero

-  read access to the location of the data

-  write access to the ManagedRepository or one of its subdirectories. More information about the ManagedRepository can be found at \ https://docs.openmicroscopy.org/latest/omero/developers/Server/FS.html


   .. image:: images/importcli2.png

**Step-by-step:**
-----------------

-  We are still in the omero server directory:
      /opt/omero/server/OMERO.server/bin

-  Log in again:
      $ ./omero --sudo importer1 -u user-1 login

-  ‘In place’ import a large SVS file into the same dataset as before:
      $ ./omero import -d $DID --transfer=ln_s
      /OMERO/in-place-import/svs/77917.svs

-  Check that the image is successfully imported.

-  Click on the paths icon |image3| to show the difference between the normal and in-place (ln_s) imported images. Validate that In-place import is indicated \ |image4|\ .

-  Note: You can use the bash script at \ https://github.com/ome/training-scripts/blob/master/maintenance/scripts/in_place_import_as.sh\  to perform the in-place import steps described above in one single command.

**Bulk Import CLI**
===================

In this example, we show how to combine several import strategies using a configuration file. This is a strategy heavily used to import data to \ https://idr.openmicroscopy.org/\ .

We import two folders named *siRNA-HeLa* and *condensation*. For this training, the path to the OMERO.server is /opt/omero/server.

1. Open a terminal and connect to the server as importer1 over SSH.

2. Note: Connecting over SSH is necessary only if you intend to import in-place. If a classic import is being performed, you can connect to the server remotely using OMERO.cli and still use the bulk import as described below.

3. Description of the files used to set up the import, the files are in the directory /OMERO/in-place-import. You can access the files on \ https://github.com/ome/training-scripts/tree/master/practical/other\ (see also \ https://docs.openmicroscopy.org/latest/omero/users/cli/import-bulk.html#bulk-imports\ for further details).

   a. import-paths.csv: (.csv, comma-separated values) this file has at least two columns. In this case the columns are separated by commas. The first column is the name of the target Dataset and the second one is the path to the folder to import. We will import two folders (the import-paths.csv has two rows).

      Example csv (note the comma between the “HeLa” and “/OMERO…”):

      *Dataset:name:Experiment1-HeLa,/OMERO/in-place-import/siRNAi-HeLa*
      
      *Dataset:name:Experiment2-condensation,/OMERO/in-place-import/condensation*

   
   b. bulk.yml: this file defines the various import options: transfer option, checksum algorithm, format of the .csv file, etc. Note that setting the dry_run option to true allows to first run an import in dry_run mode and copy the output to an external file. This is useful when running an import in parallel.
   
      Example bulk.yml:

      *continue: "true"*

      *transfer: “ln_s”*

      *# exclude: “clientpath”*

      *checksum_algorithm: “File-Size-64”*

      *logprefix: “logs”*

      *output: “yaml”*

      *path: "import-paths.csv"*

      *columns:*

          -  *target*

          -  *path*

4. Go to /OMERO/in-place-import i.e. cd /OMERO/in-place-import

5. The importer1 (Facility Manager with ability to import for others) user logs in as user-1:

   d. $ bin/omero --sudo importer1 -u user-1 login

6. Import the data using the --bulk command:

   e. $ bin/omero import --bulk bulk.yml

7. Go to the webclient during the import process to show the newly created dataset. The new datasets in OMERO are named Experiment1-HeLa and Experiment2-condensation. This was specified in the first column of the import-paths.csv file.

8. Select an image.

9. In the right-hand panel, select the General tab to validate:

   f. Click on |image3| to show the import details.

   g. Validate that In-place import is indicated \ |image4|\ .

**Advantages:**

-  Large amount of data imported using one import command.

-  Reproducible import.

**Limitations:**

-  Preparation of the .csv or .tsv file.

More information about import options using the CLI import can be found at \ https://docs.openmicroscopy.org/latest/omero/users/cli/import.html\ .

.. |image0| image:: media/image4.png
   :width: 4.46235in
   :height: 6.34896in
.. |image1| image:: media/image2.png
   :width: 6.5in
   :height: 3.65278in
.. |image3| image:: images/importcli3.png
   :width: 0.30208in
   :height: 0.21875in
.. |image4| image:: images/importcli4.png
   :width: 1.90625in
   :height: 0.31771in
