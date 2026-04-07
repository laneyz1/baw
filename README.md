from datetime import datetime, date

# ===============================
# ROOM CLASS
# ===============================
class Room:
    def __init__(self, number, capacity, room_type, course):
        self.__number = number
        self.__capacity = capacity
        self.__type = room_type
        self.__course = course

    def get_number(self):
        return self.__number

    def get_info(self, available=True):
        return f"Room {self.__number} | Cap:{self.__capacity} | {self.__type} | {self.__course} | Available:{available}"


# ===============================
# USER CLASS
# ===============================
class User:
    def __init__(self, user_id, username, password):
        self._user_id = user_id
        self._username = username
        self._password = password

    def get_username(self):
        return self._username

    def get_password(self):
        return self._password

    def check_password(self, password):
        return self._password == password

# ===============================
# ADD-ONS CLASS
# ===============================
class AddOns:
    def __init__(self, projector, chair, table, sound):
        self.projector = projector
        self.chair = chair
        self.table = table
        self.sound = sound

    def display(self):
        return f"Projector:{self.projector}, Chairs:{self.chair}, Tables:{self.table}, Sound:{self.sound}"


# ===============================
# RESERVATION CLASS
# ===============================
class Reservation:
    def __init__(self, school_id, user, room, date, start, end, addons):
        self._school_id = school_id
        self.__user = user
        self.__room = room
        self.__date = date
        self.__start = start
        self.__end = end
        self.__addons = addons
        self.__status = "Approved"

    def get_room(self):
        return self.__room
        
    def set_room(self, new_room):
        self.__room = new_room
    def get_status(self):
        return self.__status


    def get_info(self):
        return f"""
Reservee School ID: {self._school_id}
User: {self.__user.get_username()}
Room: {self.__room.get_info()}
Date: {self.__date}
Time: {self.__start}:00 - {self.__end}:00
Add-ons: {self.__addons.display()}
Status: {self.__status}
"""

    def cancel(self):

        if self.__status == "Cancelled":
            return False, "⚠️ Already cancelled."

        try:
            reservation_time = datetime.strptime(
            f"{self.__date} {self.__start}",
            "%Y-%m-%d %H"
        )
        except ValueError:
            return False, "❌ Invalid reservation date!"

        now = datetime.now()

        hours_left = (reservation_time - now).total_seconds() / 3600

        if hours_left >= 8:
            self.__status = "Cancelled"
            return True, "✅ Reservation cancelled successfully!"
        else:
            return False, "❌ Cannot cancel (less than 8 hours before reservation)."

# ===============================
# SYSTEM CLASS
# ===============================
class RoomReservationSystem:
    def __init__(self):
        self.rooms = []
        self.users = []
        self.reservations = []
        self.history = []
        self.current_user = None   

        self.load_room_data()

        # ===== DUMP USER DATA =====
        self.users.extend([
            User("U1", "faculty1", "teach2024"),
            User("U2", "student1", "stud123"),
        ])

    def load_room_data(self):
        self.rooms.extend([
            Room("101", 40, "Lecture", "BSCS"),
            Room("102", 35, "Lecture", "BSED-MATH"),
            Room("103", 40, "Lecture", "BSCS"),
            Room("104", 35, "Lecture", "BSED"),
            Room("105", 45, "Lecture", "BEED"),
            Room("106", 50, "Lecture", "BSHM"),
            Room("107", 30, "Lecture", "BTLEd-HE"),
            
            Room("201", 25, "Laboratory", "BSCS"),
            Room("202", 30, "Laboratory", "BSES"),
            Room("203", 20, "Laboratory", "BEED"),
            Room("204", 35, "Laboratory", "BTLEd-HE"),
            Room("205", 40, "Laboratory", "BSHM"),
           
            Room("301", 50, "Lecture", "BSHM"),
            Room("302", 45, "Lecture", "BSED-MATH"),
            Room("303", 50, "Lecture", "BEED"),
            Room("304", 60, "Lecture", "BSHM"),
            Room("305", 35, "Lecture", "BTLEd-HE"),

            Room("401", 20, "Laboratory", "BSES"),
            Room("402", 30, "Laboratory", "BSHM"),
            Room("403", 25, "Laboratory", "BEED"),
            Room("404", 40, "Laboratory", "BTLEd-HE"),
            Room("405", 30, "Laboratory", "BSCS"),

            Room("501", 60, "Lecture", "BSCS"),
            Room("502", 50, "Lecture", "BSED"),
            Room("503", 45, "Lecture", "BEED"),
            Room("504", 55, "Lecture", "BSHM"),
            Room("505", 40, "Lecture", "BTLEd-HE"),

        ])

    # ===========================
    # LOGIN / REGISTER
    # ===========================
    def login(self):
        print("\n====== LOGIN SYSTEM ======")

        while True:
            choice = input("Do you already have an account? (Yes/No): ").lower()

        # ===== EXISTING USER LOGIN =====
            if choice == "yes":
                username = input("Username: ").strip()
                password = input("Password: ").strip()

                for user in self.users:
                    if user.get_username() == username and user.check_password(password):
                        print("✅ Login successful!")
                        self.current_user = user
                        return

                print("❌ Invalid username or password.")

        # ===== CREATE NEW ACCOUNT =====
            elif choice == "no":
                new_username = input("Create username: ").strip()
                new_password = input("Create password: ").strip()

                user_id = f"U{len(self.users)+1}"
                new_user = User(user_id, new_username, new_password)

                self.users.append(new_user)
                self.current_user = new_user

                print("✅ Account created successfully!")
                print("✅ You are now logged in.")
                return

            else:
                print("❌ Please enter Yes or No only.")
    # ===========================
    # CHECK ROOM AVAILABILITY
    # ===========================
    def is_room_available(self, room):
        for res in self.reservations:
            if res.get_room() == room and res.get_status() == "Approved":
                return False
        return True

    # ===========================
    # USER LOGIN
    # ===========================
    def get_user(self):
        return self.current_user
        

    # ===========================
    # VIEW AVAILABLE ROOMS
    # ===========================
    def view_available_rooms(self):
        print("\n📌 AVAILABLE ROOMS:")

        available_rooms = [
            room for room in self.rooms if self.is_room_available(room)
        ]

        if not available_rooms:
            print("❌ No available rooms!")
            return []

        for i, room in enumerate(available_rooms):
            print(f"{i}. {room.get_info(True)}")

        return available_rooms

    # ===========================
    # ROOM ADD-ONS
    # ===========================
    def room_addons(self):
        print("\n📌 ROOM ITEM ADD-ONS")

        def ask(item):
            choice = input(f"Add {item}? (Yes/No): ").lower()
            if choice == "yes":
                try:
                    return int(input("Quantity: "))
                except ValueError:
                    return 0
            return 0

        return AddOns(
            ask("Projector"),
            ask("Chairs"),
            ask("Tables"),
            ask("Sound System"),
        )

    # ===========================
    # MAKE RESERVATION
    # ===========================
    def make_reservation(self):
        user = self.get_user()
        available_rooms = self.view_available_rooms()

        if not available_rooms:
            return

        # Select room
        try:
            room = available_rooms[int(input("\nSelect room index: "))]
        except (ValueError, IndexError):
            print("❌ Invalid selection!")
            return

        # Inputs
        school_id = input("\nReservee School ID: ")
        date_input = input("Date (YYYY-MM-DD): ")

        # Validate date format
        try:
            reservation_date = datetime.strptime(date_input, "%Y-%m-%d").date()
        except ValueError:
            print("❌ Invalid date format!")
            return

        # Prevent past reservations
        today = date.today()
        if reservation_date < today:
            print("❌ Invalid reservation! You cannot reserve a room in the past.")
            return

        print("✅ Reservation date accepted.")

        # Time input
        try:
            print("==== SCHOOL HOURS : 8AM - 5PM (MILITARY TIME) ====")
            start = int(input("Start hour (8-17): "))
            end = int(input("End hour (8-17): "))

            if start < 8 or end > 17 or start >= end:
                print("❌ Invalid time!")
                return

        except ValueError:
            print("❌ Invalid time input!")
            return

        # Add-ons
        addons = self.room_addons()

        # Create reservation
        reservation = Reservation(
            school_id,
            user,
            room,
            reservation_date,
            start,
            end,
            addons
        )

        # Save
        self.reservations.append(reservation)
       

        print("✅ Reservation created!")

# ===========================
# VIEW & CANCEL
# ===========================
    def view_reservations(self):
        if not self.reservations:
            print("\n⚠️ No active reservations!")
            return

        print("\n📌 ACTIVE RESERVATIONS:")

        for i, res in enumerate(self.reservations):
            print(f"{i}. {res.get_info()}")

        if input("Cancel? (Yes/No): ").lower() != "yes":
            return

        try:
            idx = int(input("Enter Reservation index: "))
            res = self.reservations[idx]

            success, msg = res.cancel()

        # ==============================
        # IF CANCEL SUCCESS
        # ==============================
            if success:
                print(msg)
                self.history.append(res)
                del self.reservations[idx]
                return

        # ==============================
        # IF CANCEL FAILED → TRANSFER ROOM
        # ==============================
            print(msg)

            current_room = res.get_room()

            for room in self.rooms:
                if (
                    room != current_room
                    and self.is_room_available(room)
                    and room._Room__type == current_room._Room__type
                    and room._Room__course == current_room._Room__course
                ):
                    res.set_room(room)
                    print(f"🔄 Reservation transferred to Room {room.get_number()} instead.")
                    return

            print("⚠️ No alternative room available for transfer.")

        except (ValueError, IndexError):
            print("❌ Invalid input!")

    # ===========================
    # HISTORY
    # ===========================
    def view_history(self):
        print("\n📌 RESERVATION HISTORY:")

        if not self.history:
            print("⚠️ No history yet!")
            return

        for i, res in enumerate(self.history):
            print(f"{i}. {res.get_info()}")

    # ===========================
    # MENU
    # ===========================
    def menu(self):
        while True:
            print("\n====== ROOM RESERVATION SYSTEM ======")
            print("1. View Available Rooms")
            print("2. Make Reservation")
            print("3. View Reservations")
            print("4. Reservation History")
            print("5. Exit")

            choice = input("Enter choice: ")

            if choice == "1":
                self.view_available_rooms()
            elif choice == "2":
                self.make_reservation()
            elif choice == "3":
                self.view_reservations()
            elif choice == "4":
                self.view_history()
            elif choice == "5":
                print("\n👋 Thank You and Godbless! Have a great dayyy!")
                break
            else:
                print("❌ Invalid choice!")


# ===============================
# RUN PROGRAM
# ===============================
if __name__ == "__main__":
    system = RoomReservationSystem()
    system.login()   
    system.menu()
