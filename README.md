
# Ex.No:1 To create a employee details fields and to display the employee details using Firebase Database in Android Studio.


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```
/*
Program to print the DatabaseTable using the firebasedatabase”.
Developed by: Naveen Kumar V
Registeration Number : 212223220068
*/
```
## Employee.java
```
package com.example.employeeapp;

public class Employee {
    private String id;
    private String name;
    private String salary;
    public Employee() {}

    public Employee(String id, String name, String salary) {
        this.id = id;
        this.name = name;
        this.salary = salary;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public String getSalary() { return salary; }

    @Override
    public String toString() {
        return name + " - ₹" + salary;
    }
}
``` 
## DBManager.java
```
package com.example.employeeapp;

import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;

public class DBManager {
    private DatabaseReference dbRef;

    public DBManager() {
        dbRef = FirebaseDatabase.getInstance().getReference("employees");
    }
    public void insertEmployee(Employee employee) {
        String key = dbRef.push().getKey();
        employee = new Employee(key, employee.getName(), employee.getSalary());
        dbRef.child(key).setValue(employee);
    }
    public void updateEmployee(Employee employee) {
        dbRef.child(employee.getId()).setValue(employee);
    }
    public void deleteEmployee(String id) {
        dbRef.child(id).removeValue();
    }

    public DatabaseReference getDatabaseRef() {
        return dbRef;
    }
}
```
## EmployeeListActivity.java
```
package com.example.employeeapp;

import android.content.Intent;
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;

import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.ValueEventListener;

import java.util.ArrayList;

public class EmployeeListActivity extends AppCompatActivity {

    private ListView listView;
    private ArrayAdapter<Employee> adapter;
    private ArrayList<Employee> employees;
    private DBManager dbManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_employee_list);

        listView = findViewById(R.id.listView);
        dbManager = new DBManager();
        employees = new ArrayList<>();

        adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, employees);
        listView.setAdapter(adapter);

        loadEmployees();
        listView.setOnItemClickListener((parent, view, position, id) -> {
            Employee selected = employees.get(position);
            Intent intent = new Intent(EmployeeListActivity.this, ModifyEmployeeActivity.class);
            intent.putExtra("id", selected.getId());
            intent.putExtra("name", selected.getName());
            intent.putExtra("salary", selected.getSalary());
            startActivity(intent);
        });
        listView.setOnItemLongClickListener((parent, view, position, id) -> {
            Employee selected = employees.get(position);
            dbManager.deleteEmployee(selected.getId());
            return true;
        });
    }

    private void loadEmployees() {
        dbManager.getDatabaseRef().addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot snapshot) {
                employees.clear();
                for (DataSnapshot ds : snapshot.getChildren()) {
                    Employee emp = ds.getValue(Employee.class);
                    if (emp != null) employees.add(emp);
                }
                adapter.notifyDataSetChanged();
            }

            @Override
            public void onCancelled(@NonNull DatabaseError error) {}
        });
    }

    public void addEmployee(android.view.View view) {
        startActivity(new Intent(this, AddEmployeeActivity.class));
    }
}
```
## AddEmployeeActivity.java
```
package com.example.employeeapp;

import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class AddEmployeeActivity extends AppCompatActivity {

    private EditText etName, etSalary;
    private DBManager dbManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_employee);

        etName = findViewById(R.id.etName);
        etSalary = findViewById(R.id.etSalary);
        dbManager = new DBManager();
    }

    public void saveEmployee(View view) {
        String name = etName.getText().toString().trim();
        String salary = etSalary.getText().toString().trim();

        if (name.isEmpty() || salary.isEmpty()) {
            Toast.makeText(this, "Enter all fields", Toast.LENGTH_SHORT).show();
            return;
        }

        dbManager.insertEmployee(new Employee(null, name, salary));
        Toast.makeText(this, "Employee Added", Toast.LENGTH_SHORT).show();
        finish();
    }
}
```
## ModifyEmployeeActivity.java
```
package com.example.employeeapp;

import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class ModifyEmployeeActivity extends AppCompatActivity {

    private EditText etName, etSalary;
    private DBManager dbManager;
    private String employeeId;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_modify_employee);

        etName = findViewById(R.id.etName);
        etSalary = findViewById(R.id.etSalary);
        dbManager = new DBManager();

        employeeId = getIntent().getStringExtra("id");
        etName.setText(getIntent().getStringExtra("name"));
        etSalary.setText(getIntent().getStringExtra("salary"));
    }

    public void updateEmployee(View view) {
        String name = etName.getText().toString().trim();
        String salary = etSalary.getText().toString().trim();

        if (name.isEmpty() || salary.isEmpty()) {
            Toast.makeText(this, "Enter all fields", Toast.LENGTH_SHORT).show();
            return;
        }

        dbManager.updateEmployee(new Employee(employeeId, name, salary));
        Toast.makeText(this, "Employee Updated", Toast.LENGTH_SHORT).show();
        finish();
    }

    public void deleteEmployee(View view) {
        dbManager.deleteEmployee(employeeId);
        Toast.makeText(this, "Employee Deleted", Toast.LENGTH_SHORT).show();
        finish();
    }
}
```
## activity_employee_list.xml
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:text="Add Employee"
        android:onClick="addEmployee"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```
## activity_add_employee.xml
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/etName"
        android:hint="Employee Name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <EditText
        android:id="@+id/etSalary"
        android:hint="Salary"
        android:inputType="number"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:text="Save"
        android:onClick="saveEmployee"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</LinearLayout>
```
## OUTPUT
## List Employee
<img width="342" height="648" alt="Screenshot 2025-09-08 104943" src="https://github.com/user-attachments/assets/f8230b9c-7ac9-4ad1-b88f-fa6d8da07ac7" />

## Update & Delete Employee
<img width="339" height="651" alt="Screenshot 2025-09-08 102833" src="https://github.com/user-attachments/assets/06aeb2a8-dc93-41ee-8a68-7ab3e4648b36" />

## Add Employee
<img width="333" height="648" alt="Screenshot 2025-09-08 102745" src="https://github.com/user-attachments/assets/19e3e548-03bb-427c-9cd3-f1d4054da71a" />

## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.
