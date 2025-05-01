# Build a fitness app Backend like Cult  Fit.

---

## 1. User Management

### Feature:
Users can register as either a user or an admin.

### Details:
- Email must be unique.
- Phone number must follow E.164 format.
- Each user has:
  - `strike_count`
  - `no_show_count`
  - Optional `banned_until` timestamp.
- Users can update their name and phone (not email).
- Deleting a user must:
  - Cancel their future bookings.
  - Remove them from waitlists.
  - Trigger waitlist promotions if needed.

### Corner Cases:
- Creating users with duplicate emails should fail.
- Deleting a waitlisted user from multiple classes must promote others correctly.

---

## 2. Authentication and Authorization

### Feature:
Secure API using JWT with role-based access control.

### Details:
- Normal users cannot access Admin APIs.
- Authorization failures must return a standardized error format.

### Corner Cases:
- Expired JWT tokens.
- Missing tokens.
- Role mismatch (e.g. user accessing admin API).

---

## 3. Center Management

### Feature:
Admins can manage gym centers.

### Details:
- Create, Update, View, Delete Centers.
- Centers cannot be deleted if active classes are scheduled.

#### Center Holidays:
- Admins can declare blackout dates.
- All class instances on blackout dates must be cancelled automatically.
- Users with bookings on those dates must receive email notifications.

### Corner Cases:
- Overlapping holidays should merge cleanly.
- Deleting a center with live bookings should be blocked.

---

## 4. Class Management (Single and Recurring)

### Feature:
Classes can be created as one-time events or recurring schedules.

### Details:
- **Single Class**: Occurs on a specific date.
- **Recurring Class**: 
  - Happens on selected weekdays.
  - Defined start and end date.
- Mandatory:
  - Class duration of 50 minutes.
  - 10-minute buffer between classes.
  - No class on center holidays.

### Corner Cases:
- Overlapping class times (even across types) must be rejected.
- Center holidays must cancel corresponding class instances.

---

## 5. Class Instance Management

### Feature:
Each scheduled date of a recurring class becomes a class instance.

### Details:
- Bookings are tied to class instances.
- Class instances on holiday dates must be deleted/cancelled.
- Late cancellations must dynamically update future instances.

### Corner Cases:
- Updating recurrence should safely regenerate future instances.
- Deleting a center must also cancel all child class instances.

---

## 6. Booking Management and Waitlist

### Feature:
Users can book spots in class instances.

### Details:
- Each instance has a capacity.
- Overbooking adds users to a waitlist.
- Booking closes 1 hour before class start.
- Users can have a maximum of 4 active bookings per week.

### Corner Cases:
- Duplicate bookings must fail.
- Overbooked users must be added to waitlist with correct position.
- Booking cancellation must promote the first waitlisted user.

---

## 7. Attendance Management

### Feature:
Users must mark themselves present within the first 5 minutes of the class.

### Details:
- No attendance marking allowed after 5 minutes.
- Missing attendance counts as a no-show.

### Corner Cases:
- Attendance attempts after 5 minutes must fail.
- Unmarked attendance triggers automatic no-show logic.

---

## 8. Cancellation Management and Penalties

### Feature:
Late cancellations and no-shows incur strikes and bans.

### Details:
- Cancel < 2 hours before class → 1 strike.
- No-show (unmarked attendance) → 1 strike.
- 3 strikes → 3-day booking ban.
- 3 no-shows → 7-day booking ban.

### Corner Cases:
- Monthly strike count reset.
- Auto-unban after penalty period ends.

---

## 9. Penalty Management APIs

### Feature:
Users and admins can view penalties.

### Details:
- Strike Count
- No-show Count
- Active Bans

---

## 10. Weekly Booking Limit Enforcement

### Feature:
Limit of 4 active bookings per user per week.

### Details:
- System must dynamically count future bookings.
- Cancelled/expired bookings free up quota.

### Corner Cases:
- Cancellations should allow immediate rebooking.
- Exceeding limit must return a clean failure.

---

## 11. Exercise Management

### Feature:
Admins can attach exercises to classes.

### Details:
- Each Exercise includes:
  - Name
  - Description
  - Difficulty level
  - Optional video URL

### Corner Cases:
- Reordering exercises must be atomic.
- Duplicate order values must be rejected.

---

## 12. Workout Logging (User Logs)

### Feature:
Users can log workout performance per class and exercise.

### Details:
- Fields:
  - Sets
  - Reps per set
  - Optional comments

### Corner Cases:
- Cannot log for unattended classes.
- Only scheduled class exercises can be logged.

---

## 13. Workout Goals and Progress Tracking

### Feature:
Users can set and track workout goals.

### Details:
- Goal includes:
  - Exercise Name
  - Target Sets and Reps
- System tracks actual vs target over time.
- User can view progress history.

### Corner Cases:
- System must auto-mark goals as *Achieved* when target is met.
- Users can delete or redefine goals.

---

## 14. No-Show Management

### Feature:
Track attendance no-shows.

### Details:
- 3 no-shows result in a 7-day ban.
- Users can view no-show history.

### Corner Cases:
- If class is later cancelled, corresponding no-show must be cleared.

---

## 15. Admin Reporting Features

### Feature:
Admins can view operational metrics.

### Details:
- **Daily Center Utilization Report:**
  - Number of classes held.
  - Occupancy percentage.
- Active holiday list view.

### Corner Cases:
- Days with no classes should be handled gracefully.
- Historical reports must remain unchanged even if holidays are added later.

---
