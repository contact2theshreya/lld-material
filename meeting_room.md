## System Requirements
The Meeting Room Scheduler should:

Handle multiple meeting rooms with defined capacities.
Allow booking a meeting room for a specific time slot and number of participants.
Notify participants about meetings and changes.
Keep track of room availability and participants’ calendars.
Support booking strategies (e.g., First-Come, First-Serve).
Manage updates or cancellations of meetings.

## Modular Entities
The system’s core entities include:

MeetingRoom: Represents rooms with capacity and booking history.
Participant: Holds participant details and their personal calendar.
Meeting: Links participants, rooms, and time slots.
TimeSlot: Represents a range of time with conflict checks.

## Design Patterns
Observer Pattern: Used for participant notifications. The Participant class implements the Observer interface, and the Meeting class manages notifications.

Strategy Pattern: Allows flexible room selection strategies. We implement a RoomSelectionStrategy interface with concrete strategies like First-Come First-Serve.

## Thread Safety
Concurrency is handled using locks for critical sections, ensuring consistent room availability and booking.

<img width="356" alt="image" src="https://github.com/user-attachments/assets/7ed8eceb-ab5b-41ac-968e-daccde736bf6" />

<img width="359" alt="image" src="https://github.com/user-attachments/assets/a8dce59d-31e7-4b87-a62d-8ee1e8c58ebb" />

## Concurrency Handling
To handle multiple users booking simultaneously, thread-safe mechanisms like synchronized blocks or ReentrantLock can be implemented.

Example in MeetingRoom:

public synchronized boolean isAvailable(TimeSlot slot) {
    return this.calendar.isAvailable(slot);
}

## MAin file
class MeetingApplication {
    public static void main(String[] args) {

        MeetingRoom room1 = new MeetingRoom("Room1", new Location(1,101), 3);
        MeetingRoom room2 = new MeetingRoom("Room2", new Location(1,101), 2);
        MeetingRoom room3 = new MeetingRoom("Room3", new Location(1,101), 5);

        MeetingRoomController controller = new MeetingRoomController();

        // Add meeting rooms
        controller.addMeetingRoom(room1);
        controller.addMeetingRoom(room2);
        controller.addMeetingRoom(room3);

        // Create participants
        Participant p1 = new Participant("ABC", "abc@gmail.com", "+91234567890");
        Participant p2 = new Participant("XYZ", "xyz@gmail.com", "+91234567890");

        List<Participant> participants = new ArrayList<>();
        participants.add(p1);
        participants.add(p2);

        // Create a meeting
        MeetingScheduler scheduler = new MeetingScheduler(new FirstComeFirstServeStrategy(), controller);
        scheduler.scheduleMeeting("M1", "Archiving Discussion", participants, new TimeSlot(2, 3));
        scheduler.scheduleMeeting("M2", "Workflow Discussion", participants, new TimeSlot(2, 3));

        // Cancel Meeting
        // scheduler.cancelMeeting(meeting);
    }

    <img width="402" alt="image" src="https://github.com/user-attachments/assets/9300916a-31d7-45c2-a59d-62091430c9e8" />

    ## Meeting room controller
    class MeetingRoomController {
    List<MeetingRoom> meetingRooms = new ArrayList<>();

    public void addMeetingRoom(MeetingRoom room) {
        this.meetingRooms.add(room);
        System.out.println("Add Rooms: " + Arrays.toString(this.meetingRooms.toArray()));
    }

    public List<MeetingRoom> getAvailableMeetingRooms(int startTime, int endTime, int numParticipants) {
        List<MeetingRoom> availableRooms = new ArrayList<>();
        for(MeetingRoom room: this.meetingRooms) {
            System.out.println("Room: " + room + " " + room.capacity + " " + numParticipants);
            if(room.capacity >= numParticipants && room.isAvailable(new TimeSlot(startTime, endTime))) {
                availableRooms.add(room);
            }
        }
        return availableRooms;
    }
}

## Calender
class TimeSlot {
    int startTime;
    int endTime;


    public TimeSlot(int startTime, int endTime) {
        this.startTime = startTime;
        this.endTime = endTime;
    }

    public boolean isOverlapping(TimeSlot slot, TimeSlot newSlot) {
        if((slot.startTime <= newSlot.startTime && slot.endTime <= newSlot.startTime) ||
                (slot.startTime >= newSlot.endTime && slot.endTime >= newSlot.endTime)) {
            return true;
        }
        return false;
    }
}

class Calendar {
    List<TimeSlot> bookings;

    public Calendar() {
        this.bookings = new ArrayList<>();
    }

    public void addTimeSlot(TimeSlot slot) {
        this.bookings.add(slot);
    }

    public void removeTimeSlot(TimeSlot slot) {
        this.bookings.remove(slot);
    }

    public boolean isAvailable(TimeSlot slot) {
        if(this.bookings.size() == 0) {
            return true;
        }
        for(TimeSlot booking: this.bookings) {
            if(slot.isOverlapping(slot, booking)) {
                return true;
            }
        }
        return false;
    }
}
## MeetingRoom

class MeetingRoom {
    String roomId;
    Meeting meeting;
    int capacity;
    Calendar calendar;
    Location location;

    public MeetingRoom(String roomId, Location location, int capacity) {
        this.roomId = STR."\{roomId}_\{location.floorId}_\{location.buildingId}";
        this.capacity = capacity;
        this.location = location;
        this.calendar = new Calendar();
    }

    public boolean isAvailable(TimeSlot slot) {
        return this.calendar.isAvailable(slot);
    }
}

## Meeting Scheduler
class MeetingScheduler {
    RoomSelectionStrategy selectionStrategy;
    MeetingRoomController roomController;
    public MeetingScheduler(RoomSelectionStrategy selectionStrategy, MeetingRoomController roomController) {
        this.roomController = roomController;
        this.selectionStrategy = selectionStrategy;
    }

    public MeetingRoom scheduleMeeting(String meetingId, String title, List<Participant> participants, TimeSlot timeSlot) {
        List<MeetingRoom> availableRooms = roomController.getAvailableMeetingRooms(timeSlot.startTime, timeSlot.endTime, participants.size());
        System.out.println("Available Rooms: " + Arrays.toString(availableRooms.toArray()));
        MeetingRoom selectedRoom = selectionStrategy.selectRoom(availableRooms);

        if(selectedRoom != null) {
            selectedRoom.meeting = new Meeting(meetingId, title, participants, timeSlot);
            selectedRoom.calendar.addTimeSlot(timeSlot);
            selectedRoom.meeting.notifyParticipants(STR."Meeting scheduled in room \{selectedRoom.roomId} from \{timeSlot.startTime} - \{timeSlot.endTime}");
            System.out.println(STR."Meeting scheduled in room \{selectedRoom.roomId} from \{timeSlot.startTime} - \{timeSlot.endTime}");
            return selectedRoom;
        } else {
            System.out.println(STR."No room available for given time slot and capacity");
            return null;
        }
    }

    // public void cancelMeeting(Meeting meeting) {}
}

