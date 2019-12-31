# CUMSUM Change Point Detection
The CUMSUM change point algorithm. The CUMSUM algorithm can be found in the articles linked [here](https://variation.com/wp-content/uploads/change-point-analyzer/change-point-analysis-a-powerful-new-tool-for-detecting-changes.pdf) or [here](https://variation.com/change-point-analysis-a-powerful-new-tool-for-detecting-changes/).

## Functions
#### change_points
The algorithm can be found in the change_points function. This function only produces one change point.
```
The CUMSUM change point algorithm.

	Args:
		*column: list(float). A single list of numeric time series data points.
		*specified_confidence_level: float (0.0,1.0). The confidence level which the change point is detected. (default 0.95)
		*niterations: int > 10. The number of bootstrapping iterations in the CUMSUM algorithms (default 100)
		*acc: boolean. If True, df_column time series points will be subtacted from their previous points, obtaining the 'velocity' of the time series. The change point detection algorithm is used on the 'velocity'. (default False)
		*rank: boolean. If True, time series points within the change point analysis are ranked by value and the algorithm is used on the ranked time series data. (default False)
	
	Returns:
		*change_point_index: int/None. if there is a change point, it returns the index where the change point is expected to be located. If a change point is not found, then None is returned.
```

#### change_point_detection
To run across a time series and obtain multiple change points, use the change_point_detection function.
```
Uses the change point algorithm to find change points in the time series data.
	
	Args:
		*df_column: list(float). A single list of numeric time series data points.
		*length_of_interval: even int. the length of the interval to search for a change point. If a change point is not found within the interval, then the algorithm searches again, this time including half the original interval. For example, if the length_of_interval=20 and len(df_column)=50, assuming there is no change point, the algorithm will search intervals 0-19, 10-29, 20-39, 30-49. If there are change points detected at points 8 and 32, the algorithm will search 0-20, 8-27, 18-37, 32-49. (default 20)
		*specified_confidence_level: float (0.0,1.0). The confidence level which the change point is detected. (default 0.95)
		*niterations: int > 10. The number of bootstrapping iterations in the CUMSUM algorithms (default 100)
		*acc: boolean. If True, df_column time series points will be subtacted from their previous points, obtaining the 'velocity' of the time series. The change point detection algorithm is used on the 'velocity'. (default False)
		*rank: boolean. If True, time series points within the change point analysis are ranked by value and the algorithm is used on the ranked time series data. (default False)
	
	Returns:
		*change_point_indices: list(int). returns a list of indices where change points are found. The last index and first index are always included as change points.
```

#### bit_depth_change
After obtaining the change points, and thus the intervals where there is no change, the bit_depth_change function will provide details whether the time series is increasing, decreasing, or constant inside each interval (between change points).
```
Once the change_point_indices are found using the change_point_detection function, this function determines whether the time series between change points is increasing, decreasing, or constant in value. 
	
	Args:
		*df_column: list(float). A single list of numeric time series data points. The same df_column used in the change_point_detection function.
		*change_point_indices: list(int). The change_point_indices returned from the change_point_detection function.
		*p_threshhold: float (0.0,1.0). The p-value determining whether the time series is increasing or decreasing. (default 0.05)
		*avg_threshhold: float > 0. If the statistic is less than the p_threshhold, then the interval is increasing or decreasing. The absolute average of the slope must be greater than the avg_threshhold to be increasing or decreasing. (default 0)
	
	Returns:
		*bit_depth_change: list(int). Returns a list the size of df_column that correlates with the slope observed at that location. If -1, the slope is decreasing. If 0, the slope is constant. If 1, the slope is increasing.
```

#### direction_smoother
Use the direction_smoother function to remove intervals (series between change points) that are too short.
```
After the bit_depth_change column is obtained from the bit_depth_change function, this function can be used to smooth any increasing, decreasing, or constant intervals too small in time. If there is an interval less that secs, half goes to the previous interval and half goes to the latter.
	
	Args:
		*direction_column: list(int). The bit_depth_change column obtained from the bit_depth_change function.
		*secs: int > 0. If there is an interval less that secs, half goes to the previous interval and half goes to the latter.
	
	Returns:
		*direction_column: list(int). Returns a list the size of the provided direction_column that correlates with the slope observed at that location. If -1, the slope is decreasing. If 0, the slope is constant. If 1, the slope is increasing. Now there are no intervals less than secs.
```

#### change_point_loop
Once the direction of each interval is found, the change_point_loop function can ensure there are no change points within increasing, descreasing, or constant intervals that may have been forgone.
```
Finds if there are any change points within the change in direction_column. For example, find if there are any changes in successive 0s, then once that turns into 1 ( or -1) find if there is any change in that.
	
	Args:
		*direction_column: list(int). The bit_depth_change column obtained from the bit_depth_change function or the direction_smoother function.
		*df_column: list(float). The original df_column used in the change_point_detection function.
		*scl: float (0.0,1.0). (specified_confidence_level)The confidence level which the change point is detected. (default 0.95)
		*niter: int > 10. (niterations)The number of bootstrapping iterations in the CUMSUM algorithms (default 100)
		*acc: boolean. If True, df_column time series points will be subtacted from their previous points, obtaining the 'velocity' of the time series. The change point detection algorithm is used on the 'velocity'. (default False)
		*rank: boolean. If True, time series points within the change point analysis are ranked by value and the algorithm is used on the ranked time series data. (default False)
		*p_thresh: float (0.0,1.0). (p_threshhold)The p-value determining whether the time series is increasing or decreasing. (default 0.05)
		*avg_thresh: float > 0. (avg_threshhold)If the statistic is less than the p_threshhold, then the interval is increasing or decreasing. The absolute average of the slope must be greater than the avg_threshhold to be increasing or decreasing. (default 0)
	
	Returns:
		*direction_column: list(int). Returns a list the size of the provided direction_column that correlates with the slope observed at that location. If -1, the slope is decreasing. If 0, the slope is constant. If 1, the slope is increasing.
```

#### final_direction
The final_direction function obtains the final direction (increasing, decreasing, constant) between each interval. This function can be after the known change points come to a convergence.
```
Make sure the directions are going in the proper position after the smoothing and the change_point_loop.
	
	Args:
		*direction_column: list(int). The bit_depth_change column obtained from the bit_depth_change function, direction_smoother function or the change_point_loop function.
		*df_column: list(float). The original df_column used in the change_point_detection function.
		*p_threshhold: float (0.0,1.0). The p-value determining whether the time series is increasing or decreasing. (default 0.05)
		*avg_threshhold: float > 0. If the statistic is less than the p_threshhold, then the interval is increasing or decreasing. The absolute average of the slope must be greater than the avg_threshhold to be increasing or decreasing. (default 0)
	
	Returns:
		*direction_column: list(int). Returns a list the size of the provided direction_column that correlates with the slope observed at that location. If -1, the slope is decreasing. If 0, the slope is constant. If 1, the slope is increasing.
```