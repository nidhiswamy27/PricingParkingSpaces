# Price graphs
![model2](https://github.com/user-attachments/assets/978dad40-5b0c-4a82-bb85-f5f76378de76)
![model1](https://github.com/user-attachments/assets/34b4807c-6d74-4da1-86ef-8ad43d3966be)

# PricingParkingSpaces
Urban parking spaces are a limited and highly demanded resource. Prices that remain static throughout the day can lead to inefficiencies — either overcrowding or underutilization. To improve utilization, dynamic pricing based on demand, competition, and real-time conditions is crucial. This project simulates such a system: an intelligent, data-driven pricing engine for parking spaces. 

# Tech Stack
numpy
pandas
matplotlib
datetime
pathway
bokeh
colab

# Architecture flow
Model1:
   A[replay_csv (parking_stream.csv)] --> B[Join with Static Table]
    B --> C[Access Previous Price]
    C --> D[Compute Demand (Occupancy / Capacity)]
    D --> E[Calculate New Price: prev_price + α × demand]
    E --> F[Update Static Table using coalesce()]
    F --> B

Model2:
  A[Start: CSV File<br>parking_stream_small.csv] --> B[Replay with Pathway<br>pw.demo.replay_csv]
    B --> C[Parse Columns & Encode<br>Location, VehicleType, TrafficLevel]
    C --> D[Add Timestamp & Day Columns<br>strptime, .dt formatting]
    D --> E[Tumbling Window Aggregation<br>30-min by Location]
    E --> F[Reduce & Compute Aggregates<br>Occupancy, Capacity, QueueLength,<br>Encoded features, Count]
    F --> G[Compute Averages<br>Occupancy / Capacity, etc.]
    G --> H[Calculate Normalized Demand<br>Weighted Sum Formula]
    H --> I[Calculate Final Price<br>BasePrice * (1 + λ * Demand)]
    I --> J[Result Table<br>Time, Location, Price]
    J --> K[Sink: compute_and_print<br>(or jsonlines/write)]

Windowed by 2 hours in Model1 and 30 mins in Model2. 
Made instances(partitions) on location so that I could independently calculate the prices for any location.
For Model 1 took alpha as 10 so that prices stayed between 10 and 20.
For Model 2, divided 10 between the factors based on how important they were in increasing demand  so that it directly came out normalised:
   5 - Occupany/Capacity being the most important factor was multiplied by 5 (always took values less than 1)
   3 - Queue Length being the next important factor was multiplied by (3/20) (took values mostly upto 20)
   1 - Vehicle Type (didn't give it much weight because no matter the size of the vehicle, a complete lot would be occupied; however this gave releif to smaller vehicles since they lead to lesser congestion) - multiplied by (1/4) (encoded to values between 1 and 4 based on size)
   0.5 - Traffic (very less weight - can lead to both increase and decrease in demand) - multiplied by (1/6) (encoded to values between 1 and 3 based on traffic level)
   0.5 - Special Day (less weight because increase in demand due to this would be reflected in other features) - multiplied by (1/2) (values were either 0 or 1)
   
