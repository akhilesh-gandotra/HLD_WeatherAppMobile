# HLD_WeatherAppMobile
HLD for designing mobile side of a weather app


# Problem statment
Design a weather app for iOS. The app should display the weather info like temperature and 7 day forcast, along with other params. The app should be glitch free, and fast.

# Requirements
- Fetch weather
- The update should be near real time
- Update should be fast
- We would need a location for update either manual or automatic

# Thought process
Weather consists of different params like temperature, railfall, wind speed etc. All of these change often but not in seconds, usually the updates are after sometime. All these updates can be recived by the fetchWeather API which requires the coordinates of the location like this `fetchWeather(coordinates: [String])` 

Whenever we launch/enter the app the app should fetch the location automatically/manually. As soon we get the location update, the above call needs to happen.

Now this call would have alot of data as there is current data, then forecast and then other conditions about the weather. So to reduce the data and payload, we can device a mechanism where only the data which has changed will be received. For this we can get the data for the first time and start caching it onwards and create its hash value as well, using some function which is shared by BE as well, when ever we call the fetchAPI we would send the checksum as well, the BE would try to find the version and if its latest there will be no updates, or if there are changes it will only send those changes with us.

The mobile app would parse the changes and update the cache with the new changes. 

The mobile app would have different columns for displaying all the data. So we will also reload the respective columns.  

With all the pros like small payload, better performance. There are certain cons with the above approach:

❌ Cons / Considerations
| Risk                                                | Mitigation                                                      |
| --------------------------------------------------- | --------------------------------------------------------------- |
| **Checksum mismatch due to serializer differences** | Agree on hashing logic with BE; canonical JSON formats          |
| **Complexity in delta parsing**                     | Use codable structs + deep merging logic                        |
| **Data corruption in cache**                        | Use integrity checks, fall back to full refresh                 |
| **New schema rollout**                              | Version delta structure; fallback to full object if unsupported |

Plus this approach is also tough to implement, lot of complex thinking and execution with chances of failure, with small benefits. 


So a simpler approach should be good, we can call `fetchWeather(coordinates: [String])` and get all the details at once, we can also split API, but having the data in one and loading at one go is good. We would need to implement caching logic as well, and this should be a timebased caching. The cache can be maintained for X minutes, where we would not call the API eg for 5 mins if the user visits again and again, the call can be suspended. After 5 minutes to 1 hr, we can show the cached data and do call in the BG. Afterwards we always call the API and show loading.

PROS:

| Area                       | Benefit                                       |
| -------------------------- | --------------------------------------------- |
| **Simple**                 | Easy to implement and debug                   |
| **Predictable UX**         | User always sees something quickly            |
| **Offline-friendly**       | Works without internet for some time          |
| **Good UX**                | Smooth refresh without spinners in most cases |
| **Battery/data efficient** | Reduces frequent polling and bandwidth        |


# Follow ups:

1. What happens if the network is lost
We can check for Reachability and also implement retry mechanism.

2. How do you handle warning updates like cyclones etc
We can have another column in the UI for such updates, everytime the API returns it can add on stuff there. Plus we can have the push notifications in case of emergencies like cyclones/heavy rains. Upon opening from such notifications, the data will always refresh

3. What happens when we want to add a widget for this.
   We can have the widget UI, and we can utilise the above logic for updating weather as well. For this we can segregate the logic of api call and the caching to a weather service layer. The widget extension methods can call the service layer for the updates. But if the widget will only require small updates like temperature and cloud info then we can have a different small API for this, but for our MVP we can have the same laer resued.
   

