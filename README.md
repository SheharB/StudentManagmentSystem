# StudentManagmentSystem

using System;

public class Program
{
    public static void Main()
    {
        // Assuming the existence of StudentDetails, CourseDetails, Course, and Attendance classes
        StudentDetails studentDetails = new StudentDetails();
        CourseDetails courseDetails = new CourseDetails();
        Course course = new Course("CourseName", 3, "TeacherName");
        Attendance attendance = new Attendance();

        bool exitProgram = false;

        do
        {
            Console.Clear();
            PrintBox("Student Management System");
            Console.WriteLine("1. Enroll Student");
            Console.WriteLine("2. Display All Students");
            Console.WriteLine("3. Add Course");
            Console.WriteLine("4. Display All Courses");
            Console.WriteLine("5. Enroll in a Course");
            Console.WriteLine("6. View Registered Courses");
            Console.WriteLine("7. Take Attendance");
            Console.WriteLine("0. Exit");
            PrintBoxSeparator();
            Console.Write("Enter your choice: ");

            if (int.TryParse(Console.ReadLine(), out int choice))
            {
                switch (choice)
                {
                    case 1:
                        studentDetails.EnrollStudent();
                        break;
                    case 2:
                        studentDetails.DisplayAllStudents();
                        break;
                    case 3:
                        courseDetails.AddCourse();
                        break;
                    case 4:
                        courseDetails.DisplayAllCourses();
                        break;
                    case 5:
                        EnrollInCourse(course);
                        break;
                    case 6:
                        ViewRegisteredCourses(studentDetails);
                        break;
                    case 7:
                        TakeAttendance(attendance);
                        break;
                    case 0:
                        exitProgram = true;
                        PrintBox("Exiting program. Goodbye!");
                        break;
                    default:
                        PrintErrorBox("Invalid choice. Please try again.");
                        break;
                }
            }
            else
            {
                PrintErrorBox("Invalid input. Please enter a number.");
            }

            if (!exitProgram)
            {
                Console.WriteLine("Press any key to return to the main menu...");
                Console.ReadKey();
            }

        } while (!exitProgram);
    }

    private static void EnrollInCourse(Course course)
    {
        Console.Write("Enter Student ID to enroll in a course: ");
        int enrollStudentId = Convert.ToInt32(Console.ReadLine());
        Console.Write("Enter Course ID to enroll in: ");
        int courseId = Convert.ToInt32(Console.ReadLine());
        course.EnrollInCourse(enrollStudentId, courseId);
    }

    private static void ViewRegisteredCourses(StudentDetails studentDetails)
    {
        Console.Write("Enter Student ID to view registered courses: ");
        int viewStudentId = Convert.ToInt32(Console.ReadLine());
        studentDetails.ViewRegisteredCourses(viewStudentId);
    }

    private static void TakeAttendance(Attendance attendance)
    {
        Console.Write("Enter Course ID for taking attendance: ");
        int courseIdForAttendance = Convert.ToInt32(Console.ReadLine());
        attendance.TakeAttendance(courseIdForAttendance);
    }

    // Helper Methods for Box Output
    private static void PrintBox(string message)
    {
        int width = message.Length + 4;
        Console.ForegroundColor = ConsoleColor.Cyan;
        Console.WriteLine(new string('=', width));
        Console.WriteLine($"| {message} |");
        Console.WriteLine(new string('=', width));
        Console.ResetColor();
    }

    private static void PrintBoxSeparator()
    {
        Console.WriteLine(new string('-', 40));
    }

    private static void PrintErrorBox(string message)
    {
        int width = message.Length + 4;
        Console.ForegroundColor = ConsoleColor.Red;
        Console.WriteLine(new string('=', width));
        Console.WriteLine($"| {message} |");
        Console.WriteLine(new string('=', width));
        Console.ResetColor();
    }
}

==================================================

public class Student
{
    public string FirstName { get; set; }
    public string LastName { get; set; }

    // Constructor
    public Student( string firstName, string lastName)
    {
        FirstName = firstName;
        LastName = lastName;
    }
}

===================================================
using System;
using System.Data.SqlClient;

public class StudentDetails
{
    private string connectionString = "Data Source=SHEHARBANO;Initial Catalog=SMS;Integrated Security=True";

    public void EnrollStudent()
    {
        Console.Write("Enter First Name: ");
        string firstName = Console.ReadLine();
        Console.Write("Enter Last Name: ");
        string lastName = Console.ReadLine();

        InsertStudentIntoDatabase(firstName, lastName);
    }

    private void InsertStudentIntoDatabase(string firstName, string lastName)
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = "INSERT INTO Students (FirstName, LastName) VALUES (@FirstName, @LastName)";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@FirstName", firstName);
                command.Parameters.AddWithValue("@LastName", lastName);

                connection.Open();
                int result = command.ExecuteNonQuery();

                if (result < 0)
                    PrintErrorBox("Error inserting data into Database!");
                else
                    PrintSuccessBox("Student enrolled successfully.");
            }
        }
    }

    public void DisplayAllStudents()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = "SELECT StudentID, FirstName, LastName FROM Students";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                connection.Open();
                using (SqlDataReader reader = command.ExecuteReader())
                {
                    PrintBox("Enrolled Students:");
                    while (reader.Read())
                    {
                        PrintBox($"ID: {reader["StudentID"]}, Name: {reader["FirstName"]} {reader["LastName"]}");
                    }
                }
            }
        }
    }

    public void ViewRegisteredCourses(int studentId)
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = "SELECT c.CourseID, c.CourseName, c.Credits " +
                           "FROM Courses c JOIN StudentEnrollments se ON c.CourseID = se.CourseID " +
                           "WHERE se.StudentID = @StudentID";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@StudentID", studentId);
                connection.Open();
                using (SqlDataReader reader = command.ExecuteReader())
                {
                    PrintBox($"Courses for Student ID {studentId}:");
                    while (reader.Read())
                    {
                        PrintBox($"ID: {reader["CourseID"]}, Name: {reader["CourseName"]}, Credits: {reader["Credits"]}");
                    }
                }
            }
        }
    }

    // Helper methods to print in boxes
    private static void PrintBox(string message)
    {
        int width = message.Length + 4;
        Console.ForegroundColor = ConsoleColor.Cyan;
        Console.WriteLine(new string('=', width));
        Console.WriteLine($"| {message} |");
        Console.WriteLine(new string('=', width));
        Console.ResetColor();
    }

    private static void PrintSuccessBox(string message)
    {
        int width = message.Length + 4;
        Console.ForegroundColor = ConsoleColor.Green;
        Console.WriteLine(new string('=', width));
        Console.WriteLine($"| {message} |");
        Console.WriteLine(new string('=', width));
        Console.ResetColor();
    }

    private static void PrintErrorBox(string message)
    {
        int width = message.Length + 4;
        Console.ForegroundColor = ConsoleColor.Red;
        Console.WriteLine(new string('=', width));
        Console.WriteLine($"| {message} |");
        Console.WriteLine(new string('=', width));
        Console.ResetColor();
    }
}

=========================================================   

public class Teacher
{
    public int TeacherID { get; set; }
    public string TeacherName { get; set; }

    // Constructor
    public Teacher(int teacherId, string teacherName)
    {
        TeacherID = teacherId;
        TeacherName = teacherName;
    }

    // Additional methods as needed
}

=========================================================

using System;
using System.Data.SqlClient;

public class CourseDetails
{
    private string connectionString = "Data Source=SHEHARBANO;Initial Catalog=SMS;Integrated Security=True";

    public int AddTeacher()
    {
        Console.Write("Enter Teacher's First Name: ");
        string firstName = Console.ReadLine();
        Console.Write("Enter Teacher's Last Name: ");
        string lastName = Console.ReadLine();

        return InsertTeacherIntoDatabase(firstName, lastName);
    }

    private int InsertTeacherIntoDatabase(string firstName, string lastName)
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = "INSERT INTO Teachers (FirstName, LastName) VALUES (@FirstName, @LastName); SELECT SCOPE_IDENTITY();";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@FirstName", firstName);
                command.Parameters.AddWithValue("@LastName", lastName);

                connection.Open();
                int teacherId = Convert.ToInt32(command.ExecuteScalar());

                if (teacherId > 0)
                    PrintSuccessBox("Teacher added successfully.");
                else
                    PrintErrorBox("Error adding the teacher!");

                return teacherId;
            }
        }
    }

    public void AddCourse()
    {
        Console.Write("Enter Course Name: ");
        string courseName = Console.ReadLine();
        Console.Write("Enter Credits: ");
        int credits = Convert.ToInt32(Console.ReadLine());

        int teacherId = AddTeacher();

        InsertCourseIntoDatabase(courseName, credits, teacherId);
    }

    private void InsertCourseIntoDatabase(string courseName, int credits, int teacherId)
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = "INSERT INTO Courses (CourseName, Credits, TeacherID) VALUES (@CourseName, @Credits, @TeacherID)";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@CourseName", courseName);
                command.Parameters.AddWithValue("@Credits", credits);
                command.Parameters.AddWithValue("@TeacherID", teacherId);

                connection.Open();
                int result = command.ExecuteNonQuery();

                if (result < 0)
                    PrintErrorBox("Error inserting data into Database!");
                else
                    PrintSuccessBox("Course added successfully.");
            }
        }
    }

    public void DisplayAllCourses()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = "SELECT c.CourseID, c.CourseName, c.Credits, t.FirstName, t.LastName FROM Courses c JOIN Teachers t ON c.TeacherID = t.TeacherID";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                connection.Open();
                using (SqlDataReader reader = command.ExecuteReader())
                {
                    // Print the "Available Courses" header with a different color
                    PrintBox("Available Courses:", ConsoleColor.Yellow);

                    while (reader.Read())
                    {
                        string courseId = reader["CourseID"].ToString();
                        string courseName = reader["CourseName"].ToString();
                        string credits = reader["Credits"].ToString();
                        string teacherName = $"{reader["FirstName"]} {reader["LastName"]}";

                        // Combine all information into a single line
                        string courseInfo = $"ID: {courseId}, Name: {courseName}, Credits: {credits}, Teacher: {teacherName}";

                        // Print the combined information in a single box
                        PrintBox(courseInfo);
                    }
                }
            }
        }
    }



    public void TakeAttendance(int courseId)
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = @"SELECT s.StudentID, s.FirstName, s.LastName 
                             FROM Students s
                             JOIN StudentEnrollments se ON s.StudentID = se.StudentID
                             WHERE se.CourseID = @CourseID";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@CourseID", courseId);
                connection.Open();
                using (SqlDataReader reader = command.ExecuteReader())
                {
                    while (reader.Read())
                    {
                        PrintBox($"ID: {reader["StudentID"]}, Name: {reader["FirstName"]} {reader["LastName"]}");
                        Console.Write("Mark Present (Y/N): ");
                        string attendance = Console.ReadLine();

                        RecordAttendance(courseId, Convert.ToInt32(reader["StudentID"]), attendance);
                    }
                }
            }
        }
    }

    private void RecordAttendance(int courseId, int studentId, string attendance)
    {
        string status = attendance.Equals("Y", StringComparison.OrdinalIgnoreCase) ? "Present" : "Absent";

        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = "INSERT INTO Attendance (CourseID, StudentID, DateAttended, Status) VALUES (@CourseID, @StudentID, @DateAttended, @Status)";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@CourseID", courseId);
                command.Parameters.AddWithValue("@StudentID", studentId);
                command.Parameters.AddWithValue("@DateAttended", DateTime.Today);
                command.Parameters.AddWithValue("@Status", status);

                connection.Open();
                int result = command.ExecuteNonQuery();

                if (result < 0)
                    PrintErrorBox("Error recording attendance!");
                else
                    PrintSuccessBox($"Attendance recorded: Student ID {studentId} - {status}");
            }
        }
    }

    private static void PrintBox(string message, ConsoleColor headerColor = ConsoleColor.Cyan)
    {
        int width = message.Length + 4;

        // Set the color for the header
        Console.ForegroundColor = headerColor;
        Console.WriteLine(new string('=', width));
        Console.WriteLine($"| {message} |");
        Console.WriteLine(new string('=', width));
        Console.ResetColor();
    }


    private static void PrintSuccessBox(string message)
    {
        int width = message.Length + 4;
        Console.ForegroundColor = ConsoleColor.Green;
        Console.WriteLine(new string('=', width));
        Console.WriteLine($"| {message} |");
        Console.WriteLine(new string('=', width));
        Console.ResetColor();
    }

    private static void PrintErrorBox(string message)
    {
        int width = message.Length + 4;
        Console.ForegroundColor = ConsoleColor.Red;
        Console.WriteLine(new string('=', width));
        Console.WriteLine($"| {message} |");
        Console.WriteLine(new string('=', width));
        Console.ResetColor();
    }
}

=============================================================

using System;
using System.Data.SqlClient;

public class Attendance
{
    private string connectionString = "Data Source=SHEHARBANO;Initial Catalog=SMS;Integrated Security=True";

    public void TakeAttendance(int courseId)
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            // Query to select all students enrolled in the specified course
            string query = @"SELECT s.StudentID, s.FirstName, s.LastName 
                         FROM Students s
                         JOIN StudentEnrollments se ON s.StudentID = se.StudentID
                         WHERE se.CourseID = @CourseID";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@CourseID", courseId);
                connection.Open();
                using (SqlDataReader reader = command.ExecuteReader())
                {
                    Console.WriteLine("=================================");
                    Console.WriteLine("|    Take Attendance for Course   |");
                    Console.WriteLine("=================================");

                    while (reader.Read())
                    {
                        int studentId = Convert.ToInt32(reader["StudentID"]);
                        string studentName = reader["FirstName"] + " " + reader["LastName"];

                        Console.WriteLine("=================================");
                        Console.WriteLine($"| ID: {studentId}, Name: {studentName} |");
                        Console.WriteLine("=================================");
                        Console.Write("Mark Present (Y/N): ");
                        string attendance = Console.ReadLine();

                        RecordAttendance(courseId, studentId, attendance);
                    }
                }
            }
        }
    }


    private void RecordAttendance(int courseId, int studentId, string attendance)
    {
        string status = attendance.Equals("Y", StringComparison.OrdinalIgnoreCase) ? "Present" : "Absent";

        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = "INSERT INTO Attendance (CourseID, StudentID, DateAttended, Status) VALUES (@CourseID, @StudentID, @DateAttended, @Status)";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@CourseID", courseId);
                command.Parameters.AddWithValue("@StudentID", studentId);
                command.Parameters.AddWithValue("@DateAttended", DateTime.Today);
                command.Parameters.AddWithValue("@Status", status);

                connection.Open();
                int result = command.ExecuteNonQuery();

                if (result < 0)
                    Console.WriteLine("Error recording attendance for Student ID " + studentId);
                else
                {
                    if (status == "Present")
                    {
                        Console.ForegroundColor = ConsoleColor.Green; // Set text color to green for present students
                        Console.WriteLine($"Attendance recorded: Student ID {studentId} - {status}");
                    }
                    else
                    {
                        Console.ForegroundColor = ConsoleColor.Red; // Set text color to red for absent students
                        Console.WriteLine($"Attendance recorded: Student ID {studentId} - {status}");
                    }

                    Console.ResetColor(); // Reset text color to default
                }
            }
        }
    }


}

========================================================================================================


using System;
using System.Data.SqlClient;

public class Course
{
    public int CourseID { get; set; }
    public string CourseName { get; set; }
    public int Credits { get; set; }
    public string TeacherName { get; set; }
    private string connectionString = "Data Source=SHEHARBANO;Initial Catalog=SMS;Integrated Security=True";

    public Course(string courseName, int credits, string teacherName)
    {
        CourseName = courseName;
        Credits = credits;
        TeacherName = teacherName;
    }

    public void AddCourse()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = "INSERT INTO Courses (CourseID, CourseName, Credits, TeacherName) VALUES (@CourseID, @CourseName, @Credits, @TeacherName)";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@CourseID", CourseID);
                command.Parameters.AddWithValue("@CourseName", CourseName);
                command.Parameters.AddWithValue("@Credits", Credits);
                command.Parameters.AddWithValue("@TeacherName", TeacherName);

                connection.Open();
                int result = command.ExecuteNonQuery();

                if (result < 0)
                    Console.WriteLine("Error adding course!");
                else
                    Console.WriteLine("Course added successfully.");
            }
        }
    }

    public void EnrollInCourse(int studentId, int courseId)
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string query = "INSERT INTO StudentEnrollments (StudentID, CourseID) VALUES (@StudentID, @CourseID)";

            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@StudentID", studentId);
                command.Parameters.AddWithValue("@CourseID", courseId);

                connection.Open();
                int result = command.ExecuteNonQuery();

                if (result < 0)
                    Console.WriteLine("Error enrolling in course!");
                else
                    Console.WriteLine("Enrolled in course successfully.");
            }
        }
    }
}

======================================================================================================
