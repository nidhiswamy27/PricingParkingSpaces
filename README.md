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

Windowed by 2 hours in Model1 and 30 mins in Model2 with instances on location so that I can independently calculate the next prices for any location.
