# Course Service - Endpoints Testing Guide

## Server Status
✅ **Course Service is running on: http://127.0.0.1:8000**

## Quick Setup for Testing in Postman

### 1. Import Collection
- Download `Course_Service_API.postman_collection.json`
- Open Postman → Click "Import" → Select the file

### 2. Test Endpoints

#### **COURSES ENDPOINTS**

**1. List All Courses**
```
GET http://127.0.0.1:8000/api/courses/
```
Expected: Returns a list of all courses (empty if no courses created yet)

---

**2. Create a Course** ⭐ (Requires Admin Header)
```
POST http://127.0.0.1:8000/api/courses/
Headers:
  - Content-Type: application/json
  - X-User-Role: ADMIN

Body (JSON):
{
  "name": "Python Basics",
  "instructor": "Dr. Johnson",
  "category": "Programming",
  "description": "Learn Python fundamentals",
  "duration_weeks": 8,
  "max_students": 25
}
```
Expected: Returns created course with ID 1

---

**3. Get Course Details**
```
GET http://127.0.0.1:8000/api/courses/1/
```
Expected: Returns the course details for course ID 1

---

**4. Update Course** ⭐ (Requires Admin Header)
```
PUT http://127.0.0.1:8000/api/courses/1/
Headers:
  - Content-Type: application/json
  - X-User-Role: ADMIN

Body (JSON):
{
  "name": "Python Advanced",
  "instructor": "Prof. Smith",
  "category": "Programming",
  "description": "Learn advanced Python",
  "duration_weeks": 10,
  "max_students": 30
}
```
Expected: Returns updated course

---

**5. Search Courses**
```
GET http://127.0.0.1:8000/api/courses/search/?name=Python
```
Query Parameters:
- `name` - Filter by course name
- `instructor` - Filter by instructor
- `category` - Filter by category
- `search` - General search (searches name, instructor, category)

---

**6. Delete Course** ⭐ (Requires Admin Header)
```
DELETE http://127.0.0.1:8000/api/courses/1/
Headers:
  - X-User-Role: ADMIN
```
Expected: Returns success message

---

#### **ENROLLMENT ENDPOINTS**

**1. Enroll Student in Course**
```
POST http://127.0.0.1:8000/api/courses/1/enroll/
Headers:
  - Content-Type: application/json

Body (JSON):
{
  "student_id": 123
}
```
Expected: Returns enrollment confirmation

---

**2. Get Course Students**
```
GET http://127.0.0.1:8000/api/courses/1/students/
```
Expected: Returns list of students enrolled in the course

---

**3. Unenroll Student from Course**
```
DELETE http://127.0.0.1:8000/api/courses/1/unenroll/
Headers:
  - Content-Type: application/json

Body (JSON):
{
  "student_id": 123
}
```
Expected: Returns success message

---

**4. List All Enrollments**
```
GET http://127.0.0.1:8000/api/enrollments/
```
Expected: Returns all enrollments in the system

---

**5. Get Enrollment Details**
```
GET http://127.0.0.1:8000/api/enrollments/1/
```
Expected: Returns details of enrollment ID 1

---

**6. Get Student's Courses**
```
GET http://127.0.0.1:8000/api/enrollments/by_student/?student_id=123
```
Query Parameters:
- `student_id` (required) - The student's ID

Expected: Returns all courses enrolled by the student

---

## Testing Sequence (Recommended)

1. **List courses** → Should be empty
2. **Create course** → Note the returned ID (usually 1)
3. **Get course details** → Verify creation
4. **Search courses** → Should find it
5. **Enroll student** → Test with student_id: 123
6. **Get course students** → Should show the enrolled student
7. **Get student's courses** → Should show course 1
8. **Update course** → Modify details
9. **Unenroll student** → Remove enrollment
10. **Delete course** → Clean up

---

## Important Notes

- **Admin Operations**: POST, PUT, DELETE require `X-User-Role: ADMIN` header
- **Student ID**: Can be any integer (doesn't need to exist in student service for testing)
- **CORS**: Set to allow requests from API Gateway (port 9090)
- **Database**: Uses SQLite by default locally (can switch to PostgreSQL)

---

## Troubleshooting

**404 Not Found**
- Course ID doesn't exist
- Check the ID returned from creation

**403 Forbidden**
- Missing or wrong `X-User-Role: ADMIN` header on POST/PUT/DELETE

**400 Bad Request**
- Missing required fields in request body
- Invalid data format

---

## Collection Variable
```
{{base_url}} = http://127.0.0.1:8000
```

Use in Postman: `{{base_url}}/api/courses/`
