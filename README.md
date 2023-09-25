# Netcdf-with-Xarray
In this tutorial, we explore using python Xarray package to process a netcdf file. The studied netcdf file can be downloaded from the study [here](https://www.nature.com/articles/s41597-021-00964-1).

## Xarray data structures
Xarray has two main data structures: Dataset and DataArray. 

A Dataset is simply a collection of DataArrays. DataArrays store the actual data that can be converted into numpy array.

```python
import xarray
import hvplot.xarray
```

```python
path=r"../Data/SoMo.ml_v1_layer1_2000.nc"
ds=xarray.open_dataset(path)
ds

```
![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/859e0461-0f95-45f9-87db-56c5317772db)




```python
#To have a display similar to ncdump
ds.info()
```

![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/209e94b9-bb0c-4b3d-bb4d-05ca7ba93762)


### Dataset items
A Dataset consists of four items: 

1- Data Variables:  
In Xarray, 'data variables' are the variables recorded in the Dataset except coordinates. In this nc file, based on Xarray terminology, there is only one variable called 'layer1' and lat, lon, and time are categorized as 'coordinates'. However, in ncdump terminology, coordinates are also considered as data variables, as shown above, so based on ncdump terminology, there are 4 data variables in this nc file.  
![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/cbf365e5-d9b0-4aed-850d-91155c468a5f)


2- Global attributes (which is a dictionary)  
```python
ds.attrs
``` 

3- Dimensions (which is a dictionary)  
![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/ab33f39d-7af9-4b11-a127-87480879c62f)
 

4- Coordinates  
![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/fc7d1f1c-ad21-459b-ab0c-fd9ac1c1e067)



**Note that 'Dimensions' specify the Dataset dimensions (e.g., 2D, 3D, 4D,...), whereas 'Coordinates' are simply DataArrays of 'Dimensions' **

## Data processing
As mentioned, for each 'data variable', there is a DataArray.  
```python
#for each data variable, there is a DatAarray that can be accessed as below:
layer1_da = ds['layer1']
layer1_da
```
![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/0b547f09-9765-479f-8518-acba4a137a5c)


For each DataArray, we can see 'coordinates' and 'attributes' specific to that DataArray:  
```python
#again we can see coordinates and attributes specific to each variable Dataarray
layer1_da.coords
```
![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/0b979d1e-d55a-49db-849d-44510afb1136)


We can convert the DataArray into numpy format by using `.data` on a DataArray:  
```python
#converting the DataArray into numpy format
layer1_np = layer1_da.data
print(type(layer1_np))
print(layer1_np.shape)
```
![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/e7800417-fe40-4731-84a0-aefc51ab06c5)


### Selection and indexing
For selection and indexing the DataArray, one method is to convert the Datarray to numpy and use numpy indexing methods. 
Alternatively, we can directly work with Datarrays and use an improved named API as descibed below.  
  
First method is to use `.isel()` method on DataArray object (not numpy array). This is a purely integer based indexing. This also works for multiple dimensions and accepts a list or slice. The important point about this API is that we do not need to worry about the order of dimensions (as we are when working with numpy arrays). Some examples of this method are:
```python
#To get all data at 100th latitude index:
# layer1_da.isel(lat=100)

# To get data at 100th latitude, 400th and -200th lon, and 10th and 11th times:
layer1_da.isel(lat=100, lon=[-200,400], time=slice(10,12))
```
![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/d4630b8c-1f00-44e5-ba43-983c8c00c853)



Second method for selection and indexing is to use `sel()` method, which works with labels and not index of data. In below example, we are selecting data for all lat and lon and for the time equal to '2000-01-11'. Here, we do not need to know the index corresponding to our desired date (as we used with `isel()`).  
```python
layer1_da.sel(time= '2000-01-11')
```

![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/c0e7685c-45fb-403a-a55a-55df2df76de2)



However, if the desired dimension has float values, like lat and lon, it is very likely that Xarray cannot find the exact values using `.sel()` and therefore throws an error. The solution is to use 'nearest' neghbor method as below:

```python
#we need to get data at lon = 64.88

# layer1_da.sel(lon=64.88) #this will make error

layer1_da.sel(lon=64.88, method='nearest') #this solves the issue

#alternatively, we an "cheat" by using a slice to select a single float
layer1_da.sel(lon=slice(64.87,64.90))
```

**Note that we can chain multiple selections. In below example, we first select all data for the first time (using time index) and then selects data for lon=64.88:
```python
layer1_da.isel(time=0).sel(lon=slice(64.87,64.90))
```

The methods `sel()` and `isel()` return the data for the exact coordinates recorded in the netcdf file. If we are interested in a coordinate not available in the netcdf file, this means we basically needs to perform an interpolation to retrieve data at the desired coordinates. Xarray facilitates this process by a method called `interp()` on DataArrays.  
```python
#here we ask to give us data at lon of 65 and 66 (which are not in the original dataset)
layer1_da.interp(lon=[65,66])
```

## Plotting
The simplest way to make a plot is to call the `.plot()` method on a DataArray. There are three types of Xarray plots based on the dimension of the retrieved data:
- when data is 1D: Xarray makes a line plot
- When data is 2D: Xarray makes a pcolormesh using matplotlib.pyplot.pcolormesh()
- When data is 3D: Xarray makes histogram (probably not useful)

Additional arguments are passed directly to the matplotlib function. For example, when the DataArray is 1D (a line plot), we can assign additional argument (color and marker, for example) of the line plot as below:
```python
DataArray.plot.line(color="purple", marker="o")
```
Note that Xarray usually uses a proper default labels/legend for the plots:
```python
layer1_da.sel(time='2000-01-01').plot(cmap='jet')

#or we can make it as as contourf plot by
#layer1_da.sel(time='2000-01-01').plot.contourf(cmap='jet')
```
![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/af3ee22a-77f7-42db-83be-dd157da74e7d)


To make an interactive plot, we can simply use `hvplot()` (instead of `plot()`):
```python
layer1_da.sel(time='2000-01-01').hvplot(cmap='jet')
```

![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/308b5413-0ebc-4f28-ab29-c9ffe64ce2ef)


And finally we can make an interactive plot with a slider to show plots for a group of values in a dimension we can use `groupby='desired dimension'` argument. Also, to make the slider more sophisticated there are some other arguments that can be used in addition to `groupby` argument.  
```python
layer1_da.hvplot(groupby='time',cmap='jet', widget_type="scrubber", widget_location='bottom')
```

![image](https://github.com/AliForghani/Netcdf-with-Xarray/assets/22843733/40bbb61d-5da0-4c2f-8f77-9afe68035813)

 See the comprehensive [plotting documentation](https://docs.xarray.dev/en/latest/user-guide/plotting.html) for details.



