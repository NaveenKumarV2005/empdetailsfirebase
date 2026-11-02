
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
## MainActivity.java
```
package com.example.myapplication;

import android.os.Bundle;
import android.text.TextUtils;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.AutoCompleteTextView;
import android.widget.LinearLayout;
import android.widget.ProgressBar;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.SearchView;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.google.android.material.button.MaterialButton;
import com.google.android.material.dialog.MaterialAlertDialogBuilder;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.android.material.textfield.TextInputEditText;
import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;
import com.google.firebase.database.ValueEventListener;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class MainActivity extends AppCompatActivity implements ItemAdapter.OnItemClickListener {

    private static final String[] CATEGORIES = {"General", "Work", "Personal", "Urgent", "Shopping"};

    private DatabaseReference itemsRef;
    private ItemAdapter adapter;
    private ProgressBar progressBar;
    private LinearLayout layoutEmpty;
    private RecyclerView recyclerView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Apply EdgeToEdge insets
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });

        // Initialize Firebase Realtime Database
        FirebaseDatabase database = FirebaseDatabase.getInstance();
        itemsRef = database.getReference("items");

        // Initialize UI components
        progressBar = findViewById(R.id.progressBar);
        layoutEmpty = findViewById(R.id.layoutEmpty);
        recyclerView = findViewById(R.id.recyclerView);
        SearchView searchView = findViewById(R.id.searchView);
        FloatingActionButton fabAdd = findViewById(R.id.fabAdd);

        // Setup RecyclerView
        adapter = new ItemAdapter(this);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        recyclerView.setAdapter(adapter);

        // FAB Click listener to create new item
        fabAdd.setOnClickListener(v -> showAddEditDialog(null));

        // SearchView listener for real-time filtering
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                adapter.filter(query);
                return true;
            }

            @Override
            public boolean onQueryTextChange(String newText) {
                adapter.filter(newText);
                return true;
            }
        });

        // Realtime listener for Read operations
        listenForRealtimeUpdates();
    }

    private void listenForRealtimeUpdates() {
        progressBar.setVisibility(View.VISIBLE);
        itemsRef.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot snapshot) {
                List<Item> items = new ArrayList<>();
                for (DataSnapshot itemSnapshot : snapshot.getChildren()) {
                    Item item = itemSnapshot.getValue(Item.class);
                    if (item != null) {
                        if (item.getId() == null || item.getId().isEmpty()) {
                            item.setId(itemSnapshot.getKey());
                        }
                        items.add(item);
                    }
                }

                // Sort items by timestamp in descending order (newest first)
                Collections.sort(items, (i1, i2) -> Long.compare(i2.getTimestamp(), i1.getTimestamp()));

                adapter.setItems(items);
                progressBar.setVisibility(View.GONE);

                if (items.isEmpty()) {
                    layoutEmpty.setVisibility(View.VISIBLE);
                    recyclerView.setVisibility(View.GONE);
                } else {
                    layoutEmpty.setVisibility(View.GONE);
                    recyclerView.setVisibility(View.VISIBLE);
                }
            }

            @Override
            public void onCancelled(@NonNull DatabaseError error) {
                progressBar.setVisibility(View.GONE);
                Toast.makeText(MainActivity.this, "Error: " + error.getMessage(), Toast.LENGTH_LONG).show();
            }
        });
    }

    // CREATE / UPDATE Dialog
    private void showAddEditDialog(Item itemToEdit) {
        View dialogView = LayoutInflater.from(this).inflate(R.layout.dialog_add_edit_item, null);

        TextView tvDialogTitle = dialogView.findViewById(R.id.tvDialogTitle);
        TextInputEditText etTitle = dialogView.findViewById(R.id.etTitle);
        AutoCompleteTextView actvCategory = dialogView.findViewById(R.id.actvCategory);
        TextInputEditText etDescription = dialogView.findViewById(R.id.etDescription);
        MaterialButton btnCancel = dialogView.findViewById(R.id.btnCancel);
        MaterialButton btnSave = dialogView.findViewById(R.id.btnSave);

        // Setup Category dropdown options
        ArrayAdapter<String> categoryAdapter = new ArrayAdapter<>(this, android.R.layout.simple_dropdown_item_1line, CATEGORIES);
        actvCategory.setAdapter(categoryAdapter);

        boolean isEditing = itemToEdit != null;
        if (isEditing) {
            tvDialogTitle.setText("Edit Item");
            etTitle.setText(itemToEdit.getTitle());
            actvCategory.setText(itemToEdit.getCategory(), false);
            etDescription.setText(itemToEdit.getDescription());
            btnSave.setText("Update");
        } else {
            tvDialogTitle.setText("Add New Item");
            actvCategory.setText(CATEGORIES[0], false);
            btnSave.setText("Add");
        }

        AlertDialog dialog = new MaterialAlertDialogBuilder(this)
                .setView(dialogView)
                .setCancelable(true)
                .create();

        btnCancel.setOnClickListener(v -> dialog.dismiss());

        btnSave.setOnClickListener(v -> {
            String title = etTitle.getText() != null ? etTitle.getText().toString().trim() : "";
            String category = actvCategory.getText() != null ? actvCategory.getText().toString().trim() : "";
            String description = etDescription.getText() != null ? etDescription.getText().toString().trim() : "";

            if (TextUtils.isEmpty(title)) {
                etTitle.setError("Title is required");
                etTitle.requestFocus();
                return;
            }

            long currentTimestamp = System.currentTimeMillis();

            if (isEditing) {
                // UPDATE
                itemToEdit.setTitle(title);
                itemToEdit.setCategory(category);
                itemToEdit.setDescription(description);
                itemToEdit.setTimestamp(currentTimestamp);

                itemsRef.child(itemToEdit.getId()).setValue(itemToEdit)
                        .addOnSuccessListener(aVoid -> {
                            Toast.makeText(MainActivity.this, "Item updated successfully", Toast.LENGTH_SHORT).show();
                            dialog.dismiss();
                        })
                        .addOnFailureListener(e -> Toast.makeText(MainActivity.this, "Failed to update: " + e.getMessage(), Toast.LENGTH_SHORT).show());
            } else {
                // CREATE
                String newId = itemsRef.push().getKey();
                if (newId != null) {
                    Item newItem = new Item(newId, title, description, category, currentTimestamp);
                    itemsRef.child(newId).setValue(newItem)
                            .addOnSuccessListener(aVoid -> {
                                Toast.makeText(MainActivity.this, "Item added successfully", Toast.LENGTH_SHORT).show();
                                dialog.dismiss();
                            })
                            .addOnFailureListener(e -> Toast.makeText(MainActivity.this, "Failed to add: " + e.getMessage(), Toast.LENGTH_SHORT).show());
                }
            }
        });

        dialog.show();
    }

    @Override
    public void onEditClick(Item item) {
        showAddEditDialog(item);
    }

    @Override
    public void onDeleteClick(Item item) {
        showDeleteConfirmation(item);
    }
    private void showDeleteConfirmation(Item item) {
        new MaterialAlertDialogBuilder(this)
                .setTitle("Delete Item")
                .setMessage("Are you sure you want to delete \"" + item.getTitle() + "\"?")
                .setPositiveButton("Delete", (dialog, which) -> {
                    itemsRef.child(item.getId()).removeValue()
                            .addOnSuccessListener(aVoid -> Toast.makeText(MainActivity.this, "Item deleted", Toast.LENGTH_SHORT).show())
                            .addOnFailureListener(e -> Toast.makeText(MainActivity.this, "Failed to delete: " + e.getMessage(), Toast.LENGTH_SHORT).show());
                })
                .setNegativeButton("Cancel", (dialog, which) -> dialog.dismiss())
                .show();
    }
}

``` 
## activity_main.xml
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="?attr/colorSurface"
    tools:context=".MainActivity">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <com.google.android.material.appbar.MaterialToolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:title="Firebase CRUD"
            app:titleTextColor="?attr/colorOnSurface" />

        <androidx.appcompat.widget.SearchView
            android:id="@+id/searchView"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginHorizontal="12dp"
            android:layout_marginBottom="8dp"
            android:background="@drawable/bg_search_view"
            app:iconifiedByDefault="false"
            app:queryHint="Search items..." />

    </com.google.android.material.appbar.AppBarLayout>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <ProgressBar
            android:id="@+id/progressBar"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:visibility="visible"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <LinearLayout
            android:id="@+id/layoutEmpty"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:orientation="vertical"
            android:visibility="gone"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent">

            <ImageView
                android:layout_width="80dp"
                android:layout_height="80dp"
                android:contentDescription="Empty list"
                android:src="@android:drawable/ic_menu_edit"
                app:tint="?attr/colorOutline" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginTop="12dp"
                android:text="No items found"
                android:textColor="?attr/colorOnSurfaceVariant"
                android:textSize="18sp"
                android:textStyle="bold" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginTop="4dp"
                android:text="Tap '+' to add your first item"
                android:textColor="?attr/colorOutline"
                android:textSize="14sp" />
        </LinearLayout>

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerView"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:clipToPadding="false"
            android:paddingVertical="8dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            tools:listitem="@layout/item_row" />

    </androidx.constraintlayout.widget.ConstraintLayout>

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fabAdd"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="20dp"
        android:contentDescription="Add item"
        android:src="@android:drawable/ic_input_add"
        app:tint="?attr/colorOnPrimaryContainer" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>

```
## OUTPUT

## List Employee
<img width="540" height="1137" alt="WhatsApp Image 2026-11-02 at 8 46 10 AM" src="https://github.com/user-attachments/assets/4646bb96-182e-42b1-a54b-2e7c862b27bc" />

## Add Employee
<img width="540" height="1137" alt="WhatsApp Image 2026-11-02 at 8 46 10 AM" src="https://github.com/user-attachments/assets/48376b07-98ce-48cd-88f4-84b3a71e9d3b" />

## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.
