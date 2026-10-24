ANSWER:

1. What goes wrong if you use the array index as :key when filtering a list?
If we use the array index as the :key, Vue may reuse the wrong component when the list is filtered or reordered.


For example, the original list is:
- Index 0: AI Talk — details panel is open.
- Index 1: Vue Workshop.
- Index 2: Football Match.
If we filter the list and AI Talk disappears, Vue may reuse the component at index 0 for Vue Workshop. The details panel's local state might remain open on the wrong event.

Solution: Use a unique, stable event ID as the key.
<EventCard
  v-for="event in visible"
  :key="event.id"
  :event="event"
/>

2. Why is visible a computed and not a function called from the template?
A computed property caches its result and recalculates only when its reactive dependencies change. A function called from the template runs whenever Vue re-renders that part of the template.

For example:
const visible = computed(() =>
  props.events.filter(event =>
    event.title.toLowerCase().includes(search.value.toLowerCase())
  )
);

When the user types in the search box, search changes, so visible recalculates and the displayed events update.

Main difference: computed caches its result until its dependencies change, while a template function may run again on every render, even if its relevant data has not changed.
3. Why does EventCard emit book instead of changing event.booked?
There are two reasons:
1. Separation of responsibilities: EventCard displays an event and reports the user's action. The parent component handles the booking logic and updates the data.
2. Avoid directly mutating props: The event is passed from a parent to EventCard as a prop. Emitting an event allows the parent to control changes and keeps data flow easier to understand and maintain.
Example:
// EventCard.vue
emit("book", props.event.id);


The parent listens for the event:
<EventCard
  :event="event"
  @book="bookEvent"
/>


The parent then handles the booking.
4. Why doesn't the page change after const { booked } = reactive(event); booked++?
JavaScript destructuring extracts the property value into a separate variable. It disconnects that variable from the reactive object's property.
const state = reactive({ booked: 0 });
const { booked } = state;

booked++;


Here, booked is a normal local variable, not a reactive reference to state.booked. Also, because it was declared with const, incrementing it causes an error.

Correct approach:
const state = reactive({ booked: 0 });

state.booked++;


Now Vue can detect the change and update the page.
5. Where does myBookings live, and why not in EventCard or EventList?
myBookings lives in the top-level parent component that manages the events and bookings, such as App.vue.

For example:
const myBookings = ref([]);


It should not live in EventCard because an individual card should not own the application's overall booking list.

It should not live only in EventList because bookings are application-level state that may be needed by other components, such as a booking summary or a “My Bookings” section.

Keeping it in the parent provides a single source of truth and allows the parent to pass data down through props and receive actions through emitted events.