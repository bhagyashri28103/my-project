# my-project
Student Attendance System
ATTENDANCE APP
public class AttendanceApp {
    private JFrame frame;
    private StudentManager studentManager;

    public AttendanceApp() {
        studentManager = new StudentManager();
        initialize();
    }

    private void initialize() {
        frame = new JFrame("Student Attendance Management System");
        frame.setBounds(100, 100, 500, 400);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLayout(new FlowLayout());

        JButton addStudentBtn = new JButton("Add Student");
        JButton markAttendanceBtn = new JButton("Mark Attendance");
        JButton viewReportBtn = new JButton("View Attendance Report");

        addStudentBtn.addActionListener(e -> addStudent());
        markAttendanceBtn.addActionListener(e -> markAttendance());
        viewReportBtn.addActionListener(e -> viewReport());

        frame.add(addStudentBtn);
        frame.add(markAttendanceBtn);
        frame.add(viewReportBtn);

        frame.setVisible(true);
    }

    private void addStudent() {
        String name = JOptionPane.showInputDialog(frame, "Enter Student Name:");
        if (name != null && !name.trim().isEmpty()) {
            studentManager.addStudent(name.trim());
            JOptionPane.showMessageDialog(frame, "Student " + name + " added successfully!");
        } else {
            JOptionPane.showMessageDialog(frame, "Name cannot be empty!");
        }
    }

    private void markAttendance() {
        List<Student> students = studentManager.getStudents();
        if (students.isEmpty()) {
            JOptionPane.showMessageDialog(frame, "No students available. Please add students first.");
            return;
        }

        JPanel panel = new JPanel(new GridLayout(students.size(), 2));
        Map<String, JComboBox<String>> comboBoxes = new HashMap<>();

        for (Student student : students) {
            panel.add(new JLabel(student.getName()));
            JComboBox<String> status = new JComboBox<>(new String[]{"Present", "Absent"});
            comboBoxes.put(student.getName(), status);
            panel.add(status);
        }

        int result = JOptionPane.showConfirmDialog(frame, panel, 
                "Mark Attendance", JOptionPane.OK_CANCEL_OPTION, JOptionPane.PLAIN_MESSAGE);

        if (result == JOptionPane.OK_OPTION) {
            Map<String, Boolean> attendanceMap = new HashMap<>();
            for (Map.Entry<String, JComboBox<String>> entry : comboBoxes.entrySet()) {
                attendanceMap.put(entry.getKey(), 
                        entry.getValue().getSelectedItem().equals("Present"));
            }
            studentManager.markAttendance(attendanceMap);
            JOptionPane.showMessageDialog(frame, "Attendance marked successfully!");
        }
    }

    private void viewReport() {
        List<Student> students = studentManager.getStudents();
        if (students.isEmpty()) {
            JOptionPane.showMessageDialog(frame, "No students available. Please add students first.");
            return;
        }

        String[] columns = {"Student Name", "Attendance Percentage"};
        Object[][] data = new Object[students.size()][2];

        for (int i = 0; i < students.size(); i++) {
            Student s = students.get(i);
            data[i][0] = s.getName();
            data[i][1] = String.format("%.2f", s.getAttendancePercentage()) + "%";
        }

        JTable table = new JTable(data, columns);
        JScrollPane scrollPane = new JScrollPane(table);
        scrollPane.setPreferredSize(new Dimension(400, 200));

        int option = JOptionPane.showConfirmDialog(frame, scrollPane, 
                "Attendance Report", JOptionPane.OK_CANCEL_OPTION, JOptionPane.PLAIN_MESSAGE);

        if (option == JOptionPane.OK_OPTION) {
            try (FileWriter writer = new FileWriter("AttendanceReport.csv")) {
                writer.write("Student Name,Attendance Percentage\n");
                for (Object[] row : data) {
                    writer.write(row[0] + "," + row[1] + "\n");
                }
                JOptionPane.showMessageDialog(frame, "Attendance report saved as AttendanceReport.csv!");
            } catch (IOException e) {
                JOptionPane.showMessageDialog(frame, "Error saving report: " + e.getMessage());
            }
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(AttendanceApp::new);
    }
}

STUDENT CLASS
public class Student {
    private String name;
    private int totalDays;
    private int presentDays;

    public Student(String name) {
        this.name = name;
        this.totalDays = 0;
        this.presentDays = 0;
    }

    public void markPresent() {
        totalDays++;
        presentDays++;
    }

    public void markAbsent() {
        totalDays++;
    }

    public double getAttendancePercentage() {
        return totalDays == 0 ? 0 : ((double) presentDays / totalDays) * 100;
    }

    public String getName() {
        return name;
    }
}
STUDENT MANAGER CLASS
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

public class StudentManager {
    private List<Student> students;

    public StudentManager() {
        students = new ArrayList<>();
    }

    public void addStudent(String name) {
        students.add(new Student(name));
    }

    public List<Student> getStudents() {
        return students;
    }

    public void markAttendance(Map<String, Boolean> attendanceMap) {
        for (Student student : students) {
            Boolean isPresent = attendanceMap.get(student.getName());
            if (isPresent != null && isPresent) {
                student.markPresent();
            } else {
                student.markAbsent();
            }
        }
    }
}

