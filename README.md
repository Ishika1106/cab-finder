# CabFinder

CabFinder is a web application that allows users to find the nearest cab in real-time. It demonstrates full-stack development skills by integrating a modern frontend with a C++ backend API. The project highlights expertise in **web development, C++ programming, REST APIs, and geolocation-based algorithms**.

---

## Project Overview

- **Purpose:** Provide users with the nearest cab instantly and visualize it on a map.
- **Frontend:** Responsive UI using **HTML, CSS, Bootstrap, and Leaflet.js**.
- **Backend:** Lightweight **C++ REST API** that calculates distances using the **Haversine formula**.
- **Skills Demonstrated:**  
  - Full-stack integration  
  - API development  
  - Geospatial calculations  
  - Frontend design with Bootstrap and map visualizations  

---

## Features

1. **Nearest Cab Detection:**  
   Calculates the closest cab using great-circle distance (Haversine formula).  

2. **Interactive Map:**  
   Shows the user’s location and nearby cabs using **Leaflet.js**.

3. **Responsive Frontend:**  
   Modern dark theme UI with mobile-friendly design.

4. **Backend API:**  
   Lightweight **C++ server** powered by [cpp-httplib](https://github.com/yhirose/cpp-httplib).

---

## Technologies Used

| Layer      | Technologies |
|------------|--------------|
| Frontend   | HTML, CSS, Bootstrap 5, Leaflet.js |
| Backend    | C++17, cpp-httplib |
| Algorithm  | Great-circle distance / Haversine formula |

---

## Backend API

- **Endpoint:** `/find_cab`  
- **Method:** GET  
- **Query Parameters:**  
  - `lat` — User’s latitude  
  - `lon` — User’s longitude  

**Response Example:**
```json
{
  "nearest_cab": "Cab_1",
  "distance_km": "3.45"
}
## Backend Code
```cpp
#include <iostream>
#include <string>
#include <vector>
#include <cmath>
#include "httplib.h"

using namespace std;
using namespace httplib;

struct Cab {
    string id;
    double latitude;
    double longitude;
};

double toRadians(double degree) {
    return degree * M_PI / 180.0;
}

double greatCircleDistance(double lat1, double lon1, double lat2, double lon2) {
    double R = 6371.0; // Radius of Earth in km
    lat1 = toRadians(lat1);
    lon1 = toRadians(lon1);
    lat2 = toRadians(lat2);
    lon2 = toRadians(lon2);

    double dlat = lat2 - lat1;
    double dlon = lon2 - lon1;

    double a = pow(sin(dlat / 2), 2) + cos(lat1) * cos(lat2) * pow(sin(dlon / 2), 2);
    double c = 2 * atan2(sqrt(a), sqrt(1 - a));
    return R * c;
}

int main() {
    vector<Cab> cabs = {
        {"Cab_1", 12.9716, 77.5946},
        {"Cab_2", 12.9352, 77.6245},
        {"Cab_3", 12.2958, 76.6394},
        {"Cab_4", 13.0827, 80.2707}
    };

    Server svr;

    svr.Get("/find_cab", [&](const Request& req, Response& res) {
        if (!req.has_param("lat") || !req.has_param("lon")) {
            res.status = 400;
            res.set_content("{\"error\":\"Missing parameters lat/lon\"}", "application/json");
            return;
        }

        double userLat = stod(req.get_param_value("lat"));
        double userLon = stod(req.get_param_value("lon"));

        string nearestCab;
        double minDistance = 1e9;

        for (auto& cab : cabs) {
            double dist = greatCircleDistance(userLat, userLon, cab.latitude, cab.longitude);
            if (dist < minDistance) {
                minDistance = dist;
                nearestCab = cab.id;
            }
        }

        string json = "{ \"nearest_cab\": \"" + nearestCab + "\", \"distance_km\": \"" + to_string(minDistance) + "\" }";
        res.set_header("Access-Control-Allow-Origin", "*");
        res.set_header("Access-Control-Allow-Methods", "GET, POST, OPTIONS");
        res.set_content(json, "application/json");
    });

    cout << "Cab Finder API running on http://localhost:8080" << endl;
    svr.listen("localhost", 8080);
}
```
## How to Run

### 1. Frontend

You can serve the frontend using a local HTTP server for proper API requests.

**Option 1: Open in Browser**
- Open `frontend/index.html` or `frontend/predict.html` directly in a browser.
- Note: Direct file opening may cause CORS issues.

**Option 2: Serve using Python HTTP Server (Recommended)**

# Open a terminal in the frontend folder
```bash
cd frontend
python3 -m http.server 5500
Then open http://localhost:5500 in your browser.
```

###2. Backend
Ensure you have a C++17 compatible compiler and cpp-httplib.h header.

# Open a separate terminal in the backend folder
```bash
cd backend
g++ -std=c++17 cab_finder.cpp -o cab_finder -pthread
./cab_finder
```
Backend will run at http://localhost:8080.

Frontend fetches nearest cab data from this API.
