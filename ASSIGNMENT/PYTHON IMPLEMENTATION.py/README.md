import data_manager
import analysis


def print_menu():
    print("\n===== FUEL CONSUMPTION ANALYSIS SYSTEM =====")
    print("1. Register Vehicle")
    print("2. Add Fuel Entry")
    print("3. Add Trip Entry")
    print("4. View Vehicle Ranking")
    print("5. Detect Abnormal Consumption Trips")
    print("6. Generate Consolidated Report")
    print("7. Exit")


def get_valid_number(prompt):
    while True:

        try:
            value = float(input(prompt))

            if value <= 0:
                raise ValueError

            return value

        except ValueError:
            print(
                "Invalid input. Please enter a positive number."
            )


def register_vehicle():

    vehicle_number = input(
        "Enter vehicle number (e.g., TN01AB1234): "
    ).strip().upper()

    if vehicle_number in data_manager.registered_vehicle_numbers:

        print(
            "Error: Vehicle number already registered."
        )

        return

    if len(vehicle_number) < 6:

        print(
            "Error: Invalid vehicle number format."
        )

        return

    vehicle_id = (
        f"V{len(data_manager.vehicle_list) + 1:03d}"
    )

    vehicle_type = input(
        "Enter vehicle type (Car/Motorcycle/Van): "
    ).strip()

    fuel_type = input(
        "Enter fuel type (Petrol/Diesel): "
    ).strip()

    data_manager.save_vehicle(
        vehicle_id,
        vehicle_number,
        vehicle_type,
        fuel_type
    )

    print(
        f"Vehicle registered successfully with ID {vehicle_id}."
    )


def add_fuel_entry():

    vehicle_id = input(
        "Enter Vehicle ID: "
    ).strip().upper()

    date = input(
        "Enter date (YYYY-MM-DD): "
    ).strip()

    quantity = get_valid_number(
        "Enter fuel quantity (litres): "
    )

    price = get_valid_number(
        "Enter fuel price per litre (Rs.): "
    )

    data_manager.save_fuel_entry(
        vehicle_id,
        date,
        quantity,
        price
    )

    cost = analysis.calculate_fuel_cost(
        quantity,
        price
    )

    print(
        f"Fuel entry recorded. Estimated cost: Rs. {cost}"
    )


def add_trip_entry():

    trip_id = (
        f"T{len(data_manager.load_trips()) + 1:03d}"
    )

    vehicle_id = input(
        "Enter Vehicle ID: "
    ).strip().upper()

    date = input(
        "Enter date (YYYY-MM-DD): "
    ).strip()

    distance = get_valid_number(
        "Enter distance travelled (km): "
    )

    fuel_used = get_valid_number(
        "Enter fuel used (litres): "
    )

    data_manager.save_trip_entry(
        trip_id,
        vehicle_id,
        date,
        distance,
        fuel_used
    )

    mileage = analysis.calculate_mileage(
        distance,
        fuel_used
    )

    print(
        f"Trip recorded. Mileage for this trip: {mileage} km/L"
    )


def view_ranking():

    trips = data_manager.load_trips()

    summary = analysis.vehicle_summary(
        data_manager.vehicle_list,
        trips
    )

    ranking = analysis.rank_vehicles(
        summary
    )

    print("\nVehicle Efficiency Ranking:")

    if not ranking:
        print("No vehicle data available.")
        return

    for rank, (vid, mileage) in enumerate(
        ranking,
        start=1
    ):

        print(
            f"{rank}. {vid} - {mileage} km/L"
        )


def view_abnormal_trips():

    trips = data_manager.load_trips()

    summary = analysis.vehicle_summary(
        data_manager.vehicle_list,
        trips
    )

    abnormal = analysis.detect_abnormal_trips(
        trips,
        summary
    )

    print(
        "\nAbnormal / High-Consumption Trips:"
    )

    if not abnormal:

        print(
            "No abnormal trips detected."
        )

        return

    for trip in abnormal:

        print(
            f"Trip {trip['trip_id']} "
            f"(Vehicle {trip['vehicle_id']}): "
            f"{trip['calculated_mileage']} km/L"
        )


def generate_report():

    trips = data_manager.load_trips()

    analysis.generate_report(
        data_manager.vehicle_list,
        trips
    )

    print(
        "Report generated successfully as report.txt"
    )


def main():

    data_manager.load_vehicles()

    while True:

        print_menu()

        choice = input(
            "Enter your choice (1-7): "
        ).strip()

        if choice == "1":
            register_vehicle()

        elif choice == "2":
            add_fuel_entry()

        elif choice == "3":
            add_trip_entry()

        elif choice == "4":
            view_ranking()

        elif choice == "5":
            view_abnormal_trips()

        elif choice == "6":
            generate_report()

        elif choice == "7":

            print(
                "Exiting the Fuel Consumption Analysis System. Goodbye!"
            )

            break

        else:

            print(
                "Invalid choice. Please select an option between 1 and 7."
            )


if __name__ == "__main__":
    main()
