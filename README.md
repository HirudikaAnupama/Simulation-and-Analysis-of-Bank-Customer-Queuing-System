# Simulation and Analysis of Customer Queuing System in a Bank

## Overview

This project simulates and analyzes the customer queuing system in a bank using the R package **simmer**. The bank currently operates with two service counters (tellers) and faces long queues and increased waiting times due to high customer footfall. The simulation explores two scenarios:  
- **Current System**: 2 service counters  
- **Proposed System**: 3 service counters (added if average waiting time exceeds 15 minutes)

**Objectives:**

- Formulate the bank’s queuing system as an M/M/2 and M/M/3 model.
- Simulate the system over an 8-hour workday (480 minutes).
- Calculate key performance metrics:
  - Average queue length
  - Average waiting time
  - Server utilization  
- Assess the impact of adding an extra counter on overall efficiency.
- Visualize the results using histograms and line charts.
- Provide recommendations for the bank’s operational strategy.

## Problem Statement

The bank operates under the following conditions:

- **Service Counters:** 2 service counters (servers) in the current system.
- **Customer Arrival:** Follow a Poisson distribution with an average rate of 10 customers per hour (approximately 0.167 customers per minute).
- **Service Time:** Follow an exponential distribution with an average service time of 5 minutes (service rate = 1/5 per minute).
- **Queue Discipline:** Customers are served on a first-come, first-served (FCFS) basis.
- **Additional Scenario:** If the average waiting time exceeds 15 minutes, evaluate the performance of adding a third teller.

## Methodology

### Model Development

1. **Assumptions and Distributions:**
   - **Arrival Process:** Customers arrive randomly based on a Poisson process.
   - **Service Process:** Service times are exponentially distributed.
   - **Queuing Model:** 
     - Current: M/M/2 (2 servers)  
     - Proposed: M/M/3 (3 servers)
  
2. **Simulation Setup:**
   - **Time Horizon:** 480 minutes (8 hours).
   - **Trajectory Definition:** Each customer:
     1. Seizes a server.
     2. Waits for service (if all servers are busy).
     3. Is served for a random duration (exponential distribution).
     4. Releases the server.
  
3. **Implementation:** The simulation is implemented using the [simmer](https://cran.r-project.org/web/packages/simmer/index.html) package, with additional plotting packages (`simmer.plot`, `ggplot2`, and `gridExtra`) for visual analysis.

### Code Implementation

Below is the complete R code to reproduce the simulation:

```r
# Load required libraries
library(simmer)       # Core simulation engine
library(simmer.plot)  # Extract simulation results
library(ggplot2)      # Visualization
library(gridExtra)    # Arrange multiple plots

# Set seed for reproducibility
set.seed(123)

# Define parameters
customer_arrival_rate <- 10 / 60  # 10 customers per hour
customer_service_rate <- 1 / 5    # Average service time = 5 minutes
bank_open_duration <- 480         # 8 hours in minutes

# Define customer trajectory
customer_trajectory <- trajectory("Customer Service Flow") %>%
  seize("teller_counter") %>%                           # Seize a server
  timeout(function() rexp(1, customer_service_rate)) %>%  # Simulate service time
  release("teller_counter")                             # Release the server

## --- Simulation for 2 Servers (Current Scenario) ---

# Create simulation environment for 2 counters
bank_sim_two_counters <- simmer("Bank with 2 Tellers") %>%
  add_resource("teller_counter", capacity = 2) %>%
  add_generator("customer", customer_trajectory, function() rexp(1, customer_arrival_rate)) %>%
  run(until = bank_open_duration)

# Extract monitoring data
arrival_log_two_counters <- get_mon_arrivals(bank_sim_two_counters)
resource_log_two_counters <- get_mon_resources(bank_sim_two_counters)

# Calculate waiting time for each customer
arrival_log_two_counters$waiting_time <- with(arrival_log_two_counters,
  end_time - start_time - activity_time
)

# Calculate key metrics for 2 tellers
avg_wait_time_two_counters <- mean(arrival_log_two_counters$waiting_time)
avg_queue_length_two <- mean(resource_log_two_counters$queue)
server_utilization_two <- sum(resource_log_two_counters$server) / (bank_open_duration * 2)

cat("----- 2 SERVICE TELLERS -----\n")
cat("Average Wait Time:", round(avg_wait_time_two_counters, 2), "minutes\n")
cat("Average Queue Length:", round(avg_queue_length_two, 2), "customers\n")
cat("Server Utilization:", round(server_utilization_two * 100, 2), "%\n\n")

## --- Simulation for 3 Servers (Proposed Scenario) ---

# Create simulation environment for 3 counters
bank_sim_three_counters <- simmer("Bank with 3 Tellers") %>%
  add_resource("teller_counter", capacity = 3) %>%
  add_generator("customer", customer_trajectory, function() rexp(1, customer_arrival_rate)) %>%
  run(until = bank_open_duration)

# Extract monitoring data
arrival_log_three_counters <- get_mon_arrivals(bank_sim_three_counters)
resource_log_three_counters <- get_mon_resources(bank_sim_three_counters)

# Calculate waiting time for each customer
arrival_log_three_counters$waiting_time <- with(arrival_log_three_counters,
  end_time - start_time - activity_time
)

# Calculate key metrics for 3 tellers
avg_wait_time_three_counters <- mean(arrival_log_three_counters$waiting_time)
avg_queue_length_three <- mean(resource_log_three_counters$queue)
server_utilization_three <- sum(resource_log_three_counters$server) / (bank_open_duration * 3)

cat("----- 3 SERVICE TELLERS -----\n")
cat("Average Wait Time:", round(avg_wait_time_three_counters, 2), "minutes\n")
cat("Average Queue Length:", round(avg_queue_length_three, 2), "customers\n")
cat("Server Utilization:", round(server_utilization_three * 100, 2), "%\n")

## --- Visualization ---

# Server utilization plots for 2 and 3 counters
server_utilizatio_two <- plot(resource_log_two_counters,
                                metric = "usage", items = "server", step = TRUE) +
  ggtitle("Server Utilization (2 Counters)") +
  xlab("Time (min)") +
  ylab("Utilization")

server_utilizatio_three <- plot(resource_log_three_counters,
                                  metric = "usage", items = "server", step = TRUE) +
  ggtitle("Server Utilization (3 Counters)") +
  xlab("Time (min)") +
  ylab("Utilization")

grid.arrange(server_utilizatio_two, server_utilizatio_three, ncol = 2, widths = c(30, 30))

# Customer waiting time plots for 2 and 3 counters
waiting_times_two <- plot(arrival_log_two_counters,
                          metric = "waiting_time") +
  ggtitle("Customer Waiting Times (2 Counters)") +
  xlab("Waiting Time (min)") +
  ylab("Customer Index")

waiting_times_three <- plot(arrival_log_three_counters,
                            metric = "waiting_time") +
  ggtitle("Customer Waiting Times (3 Counters)") +
  xlab("Waiting Time (min)") +
  ylab("Customer Index")

grid.arrange(waiting_times_two, waiting_times_three, ncol = 2)

# Histograms of waiting times
waiting_times_two_hist <- ggplot(arrival_log_two_counters, aes(x = waiting_time)) +
  geom_histogram(bins = 30) +
  ggtitle("Waiting Times (2 Counters)") +
  xlab("Waiting Time (min)") +
  ylab("Count of Customers") +
  theme_minimal()

waiting_times_three_hist <- ggplot(arrival_log_three_counters, aes(x = waiting_time)) +
  geom_histogram(bins = 30) +
  ggtitle("Waiting Times (3 Counters)") +
  xlab("Waiting Time (min)") +
  ylab("Count of Customers") +
  theme_minimal()

grid.arrange(waiting_times_two_hist, waiting_times_three_hist, ncol = 2, widths = c(30, 31))
