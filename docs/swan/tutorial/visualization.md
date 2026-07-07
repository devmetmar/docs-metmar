# Visualization of SWAN output

## Time Series Data

Time series data output is extracted using command TABLE and in table format. The first few lines of the file contain header, followed by the data values for each variable at each time step. Users can use software like **Excel, Python, or Matlab** to read and analyze this time series data.

For example below is a line from SWAN namelist= <br>
```POINTs 'Liku' 125.58 1.654 <br>
Table 'Liku' HEAD 'L40_Likupang.tbl' TIME XP YP HS HSWELL DEPTH TM01 DIR OUTPUT 20210109.000000 30 MIN```

Points named 'Liku' with coordinates (125.58 E, 1.654 N) are defined in the namelist. The output data for this point is saved in a table file named 'L40_Likupang.tbl' with variables such as time, coordinates, significant wave height (HS), swell wave height (HSWELL), depth, mean wave period (TM01), and wave direction (DIR) at 30-minute intervals starting from 00 UTC on January 9, 2021. 

## Spatial Data

Spatial data output is extracted using command BLOCK and in grid format. The first few lines of the file contain header, followed by the data values for each variable at each grid point for each time step. Users can use software like **Python or Matlab** to read and analyze this spatial data.

For example below is a line from SWAN namelist= <br>
```BLOCK 'COMPGRID' NOHEAD 'L40_BASE.mat' LAY 1 HS HSWELL DIR TM01 OUTPUT 20210112.000000 1 HR```

A spatial output from the computational grid named 'COMPGRID' is defined in the namelist. The output data for this grid is saved in a file named 'L40_BASE.mat' with variables such as significant wave height (HS), swell wave height (HSWELL), wave direction (DIR), and mean wave period (TM01) at 1-hour intervals starting from 00 UTC on January 12, 2021.

