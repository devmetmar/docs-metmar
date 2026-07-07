# Data Preparation

As previously mentioned in the [Introduction](../overview/introduction.md) section, the SWAN model requires several input data to run a simulation. And the data must be prepared following the SWAN model requirements. Below are the details of each input data and how to prepare them:

1. **Wind Fields** <br>
   To prepare the wind data for SWAN, users may need to arrange the data to match SWAN's input requirements. Python or Matlab can be used for this purpose. Data format should be provided under format of .txt, .dat, or .in. <br>
   SWAN requires wind data (u and v) to be arranged side by side in every time step, and user have two choices to arrange the data: <br>
   a. Arrange the data in a ==single file== that contains all time steps. The data should be organized in columns, with each column representing a specific variable (e.g., u-component, v-component) at each grid point.  <br>
   b. Arrange the data in ==multiple files==, with each file representing a specific time step. Each file should contain the wind data (u and v) for all grid points at that time step, organized in columns as described above. The files should be named sequentially to indicate the time step (e.g., wind_0001.txt, wind_0002.txt, etc.). In this mode, user need to create an additional file that contains the list of all the wind files and their corresponding time steps. <br>
   
2. **Bathymetry** <br>
   To prepare the bathymetry data for SWAN, users need to ensure that the data is in the correct format and resolution. The bathymetry data should cover the entire model domain and be provided in a grid format compatible with SWAN (e.g., fort.14 format for unstructured grids). Python or Matlab can be used to convert and format the bathymetry data as needed. If the bathymetry data is downloaded from GEBCO, users need to download it ASCII and be able to edit it using Notepad. User need to remove the details of data (usually several rows from above). <br>

3. **Boundary Conditions** <br>
   To prepare the boundary conditions data for SWAN, user may use TPAR file which contains about relevant wave parameters (e.g., significant wave height, peak period, mean wave direction and directional spread) from the source data (e.g. ERA5). User must provide the data along the open boundaries of the model domain. Python or Matlab can be used to extract and format the boundary conditions data as needed. The data should be organized in a way that SWAN can read, typically in a text file format with specific columns for each variable at each boundary point. <br>

4. **Initial Conditions** <br>
   To prepare the initial conditions data for SWAN, users may use previous SWAN output files (hotfile) or other wave model outputs that cover the entire model domain at the start of the simulation period. So user need to run a spin-up simulation to generate initial conditions, which usually takes a week or two, prior the desired simulation period. <br>


   