---
label: Java Examples
icon: <svg xmlns="http://www.w3.org/2000/svg" enable-background="new 0 0 24 24" viewBox="0 0 24 24" id="java"><path d="M4.44,22.324c9.472,1.553,17.265-0.7,14.808-1.82c0,0,0.671,0.559-0.738,0.991c-2.681,0.822-11.16,1.069-13.514,0.033c-0.846-0.373,0.742-0.89,1.24-0.998c0.521-0.114,0.819-0.093,0.819-0.093C6.111,19.766,0.966,21.754,4.44,22.324z"></path><path d="M20.75 21.726c0 0-.299.775-3.532 1.391-3.646.694-8.146.613-10.813.168 0-.001.547.457 3.355.639C14.033 24.201 20.595 23.771 20.75 21.726zM15.195 16.549c-2.746.535-4.332.517-6.341.308-1.552-.163-.536-.924-.536-.924-4.019 1.348 2.237 2.879 7.852 1.218C15.574 16.938 15.195 16.549 15.195 16.549zM17.058 17.584c-.02.056-.089.117-.089.118 5.946-1.581 3.76-5.573.917-4.562-.25.09-.381.297-.381.297s.158-.064.509-.138C19.452 12.995 21.51 15.244 17.058 17.584zM9.243 7.627c-1.106 1.681.544 3.486 2.792 5.539-.877-2.004-3.851-3.759.001-6.836C16.84 2.494 14.374 0 14.374 0 15.37 3.962 10.868 5.158 9.243 7.627z"></path><path d="M17.347 4.902c0-.001-8.123 2.051-4.243 6.573 1.145 1.333-.301 2.533-.301 2.533s2.906-1.518 1.571-3.418C13.127 8.818 12.171 7.938 17.347 4.902zM8.887 18.56c-3.648 1.031 2.219 3.162 6.865 1.149-.76-.3-1.306-.646-1.306-.646-2.071.398-3.032.429-4.913.211C7.98 19.094 8.887 18.56 8.887 18.56z"></path><path d="M15.992,14.67c0.456-0.315,1.086-0.587,1.086-0.587s-1.792,0.325-3.577,0.477c-2.184,0.185-4.529,0.221-5.705,0.062C5.01,14.246,9.323,13.21,9.323,13.21s-1.675-0.114-3.733,0.892C3.153,15.293,11.614,15.835,15.992,14.67z"></path></svg>
order: 3
---

Here we discuss about an Android todo application using Nitrite database. It uses nitrite as a file based storage engine. It demonstrates the use of Nitrite database in an Android application. The full source code is available [here](https://github.com/nitrite/nitrite-android-demo). This tutorial assumes that you have basic knowledge of Android development.

## Setup

To use Nitrite in your Android application, you need to add the following dependency in your app's `build.gradle` file.

```groovy
dependencies {
    implementation platform('org.dizitart:nitrite-bom:[latest-version]')
    implementation 'org.dizitart:nitrite'
    implementation 'org.dizitart:nitrite-mvstore-adapter'
}
```

Also add these pro-guard rules in your `proguard-rules.pro` file.

```proguard
-keep class org.dizitart.no2.** { *; }
-keep class org.slf4j.** { *; }
-keep class org.h2.** { *; }
-keep class org.objenesis.** { *; }
-keep class com.fasterxml.jackson.** { *; }

-keepnames class * implements java.io.Serializable

-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}
```

## Entity Classes

Define a simple `Todo` entity class to hold your todo data:

```java
import org.dizitart.no2.collection.Document;
import org.dizitart.no2.common.mapper.EntityConverter;
import org.dizitart.no2.common.mapper.NitriteMapper;
import org.dizitart.no2.repository.annotations.Entity;

@Entity
public class TodoItem {
    @Id
    private Integer id;
    private String taskName;
    private Boolean status;
    private String color;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getTaskName() {
        return taskName;
    }

    public void setTaskName(String taskName) {
        this.taskName = taskName;
    }

    public Boolean getStatus() {
        return status;
    }

    public void setStatus(Boolean status) {
        this.status = status;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public static class Converter implements EntityConverter<TodoItem> {

        @Override
        public Class<TodoItem> getEntityType() {
            return TodoItem.class;
        }

        @Override
        public Document toDocument(TodoItem entity, NitriteMapper nitriteMapper) {
            return Document.createDocument()
                    .put("id", entity.getId())
                    .put("taskName", entity.getTaskName())
                    .put("status", entity.getStatus())
                    .put("color", entity.getColor());
        }

        @Override
        public TodoItem fromDocument(Document document, NitriteMapper nitriteMapper) {
            TodoItem item = new TodoItem();
            item.id = document.get("id", Integer.class);
            item.taskName = document.get("taskName", String.class);
            item.status = document.get("status", Boolean.class);
            item.color = document.get("color", String.class);
            return item;
        }
    }
}
```

And, a list of `TodoItem` can be represented as:

```java
public class TodoList {
    private String name;
    private String color;

    public TodoList(String name, String color) {
        this.name = name;
        this.color = color;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }
}
```

## Database Initialization

To initialize the database and to maintain a singleton instance of the database, you need to create a singleton class `NitriteManager` as follows:

```java
public class NitriteManager {
    private static NitriteManager _instance = new NitriteManager();
    private static final String dbName = "task.db";
    private static Nitrite db;

    private NitriteManager() {}

    public static NitriteManager getInstance() {
        return _instance;
    }

    private void openDb(Context context) {
        if (db != null) {
            db.close();
        }

        MVStoreModule storeModule = MVStoreModule.withConfig()
                .filePath(context.getDatabasePath(dbName))
                .build();
        SimpleDocumentMapper documentMapper = new SimpleDocumentMapper();
        documentMapper.registerEntityConverter(new TodoItem.Converter());

        db = Nitrite.builder()
                .loadModule(storeModule)
                .loadModule(() -> setOf(documentMapper))
                .openOrCreate();
    }

    public ObjectRepository<TodoItem> getTodoRepository(Context context, String label) {
        if (db == null) {
            openDb(context);
        }

        return db.getRepository(TodoItem.class, label);
    }
}
```

Next create a data access layer `DataBaseManager` for CRUD operations:

```java
public class DataBaseManager {

    private final NitriteManager nitriteManager;
    private final String label;

    public DataBaseManager(String label) {
        this.nitriteManager = NitriteManager.getInstance();
        this.label = label;
    }

    public void addTask(Context context, int id, boolean status, String taskName, String taskColor) {
        TodoItem todoItem = new TodoItem();
        todoItem.setId(id);
        todoItem.setColor(taskColor);
        todoItem.setStatus(status);
        todoItem.setTaskName(taskName);
        nitriteManager.getTodoRepository(context, label).insert(todoItem);
    }

    public void removeTask(Context context, int id) {
        nitriteManager.getTodoRepository(context, label).remove(where("id").eq(id));
    }

    public void updateTask(Context context, Boolean taskStatus, int id, String taskName) {
        Document updateDocument = Document.createDocument()
                .put("status", taskStatus)
                .put("taskName", taskName);
        nitriteManager.getTodoRepository(context, label).update(where("id").eq(id), updateDocument);
    }

    public List<TodoItem> readFromDB(Context context) {
        return nitriteManager.getTodoRepository(context, label).find().toList();
    }

    public int getCurrentBiggestId(Context context) {
        TodoItem item = nitriteManager.getTodoRepository(context, label).find(
                FindOptions.orderBy("id", SortOrder.Descending)).firstOrNull();
        if (item == null) return 0;
        return item.getId();
    }
}
```

## Activity

Now you need to create two activities, one for the todo list and another for the menu.

### Todo List Activity

```java
public class ToDoListActivity extends AppCompatActivity {

    DataBaseManager manager;
    ListView listView;
    CustomAdapter customAdapter;
    TextView activityTitle;
    EditText addTask;
    int currentBiggerId = 0;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_todo_list);
        //naming the activity
        activityTitle = (TextView) findViewById(R.id.activityTitle);
        activityTitle.setText(getIntent().getStringExtra("listName"));
        //getting list color
        String color = getIntent().getStringExtra("listColor");

        addTask = (EditText) findViewById(R.id.editTextAddTask);

        listView = (ListView) findViewById(R.id.listTodo);
        manager = new DataBaseManager(activityTitle.getText().toString());
        listView.setOnItemLongClickListener((parent, view, position, id) -> {
            Log.e("Item Long Click", "Removing the item");
            manager.removeTask(getApplicationContext(), (customAdapter.todoList.get(position)).getId());
            try {
                printDB(true);
            } catch (ExecutionException | InterruptedException e) {
                e.printStackTrace();
            }
            return false;
        });
        try {
            printDB(true);
        } catch (ExecutionException | InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void addTask(View v) {
        Runnable runnable = () -> {
            currentBiggerId = manager.getCurrentBiggestId(getApplicationContext());
            int randomInt = new Random().nextInt(3);
            String color = "";
            switch (randomInt) {
                case 0:
                    color = "blue";
                    break;
                case 1:
                    color = "green";
                    break;
                case 2:
                    color = "violet";
                    break;
            }
            manager.addTask(getApplicationContext(), new Random().nextInt(), false, addTask.getText().toString(), color);
            try {
                printDB(true);
            } catch (ExecutionException | InterruptedException e) {
                e.printStackTrace();
            }
        };
        runnable.run();
    }

    public void printDB(boolean updateFromDb) throws ExecutionException, InterruptedException {
        if (updateFromDb) {
            customAdapter = new CustomAdapter(manager.readFromDB(getApplicationContext()));
            listView.setAdapter(customAdapter);
        }
        customAdapter.notifyDataSetChanged();
        currentBiggerId = manager.getCurrentBiggestId(getApplicationContext());
        addTask.setText("");
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();
        if (id == R.id.about) {
            Toast.makeText(this, "Nitrite Database Demo", Toast.LENGTH_LONG).show();
        }
        return super.onOptionsItemSelected(item);
    }

    @Override
    protected void onResume() {
        super.onResume();
        try {
            printDB(true);
        } catch (ExecutionException | InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(customAdapter.todoList.size());
    }

    public class CustomAdapter extends BaseAdapter {
        List<TodoItem> todoList;

        CustomAdapter(List<TodoItem> todoList) {
            this.todoList = todoList;
        }

        @Override
        public int getCount() {
            return todoList.size();
        }

        @Override
        public Object getItem(int i) {
            return null;
        }

        @Override
        public long getItemId(int i) {
            return i;
        }

        @Override
        public View getView(final int i, View view, ViewGroup viewGroup) {
            view = getLayoutInflater().inflate(R.layout.task_todo_item, null);

            final TextView title = view.findViewById(R.id.todoName);
            title.setText(todoList.get(i).getTaskName());
            CheckBox button = view.findViewById(R.id.checkButton);
            button.setChecked(todoList.get(i).getStatus());

            //region setting color states
            int[][] states = new int[][]{
                    new int[]{-android.R.attr.state_checked}, //disabled
                    new int[]{android.R.attr.state_checked} //enabled
            };
            @SuppressLint("ResourceType") int[] colorBlue = new int[]{Color.parseColor(getString(R.color.colorBlue)),
                    Color.parseColor(getString(R.color.colorBlue))};
            @SuppressLint("ResourceType") int[] colorGreen = new int[]{Color.parseColor(getString(R.color.colorGreen)),
                    Color.parseColor(getString(R.color.colorGreen))};
            @SuppressLint("ResourceType") int[] colorViolet = new int[]{Color.parseColor(getString(R.color.colorViolet)),
                    Color.parseColor(getString(R.color.colorViolet))};
            //endregion

            //region setting UI colors
            RelativeLayout layout = view.findViewById(R.id.itemBackground);
            String color = todoList.get(i).getColor();
            final int taskId = todoList.get(i).getId();
            final String taskName = todoList.get(i).getTaskName();
            if (color.matches("blue")) {
                layout.setBackground(ContextCompat.getDrawable(getApplicationContext(), R.drawable.list_item_background_blue));
                button.setButtonTintList(new ColorStateList(states, colorBlue));
            } else if (color.matches("green")) {
                layout.setBackground(ContextCompat.getDrawable(getApplicationContext(), R.drawable.list_item_background_green));
                button.setButtonTintList(new ColorStateList(states, colorGreen));
            } else {
                layout.setBackground(ContextCompat.getDrawable(getApplicationContext(), R.drawable.list_item_background_violet));
                button.setButtonTintList(new ColorStateList(states, colorViolet));
            }
            //endregion
            button.setOnCheckedChangeListener((compoundButton, b) -> itemChecked(b, taskId, taskName));

            return view;
        }

        void itemChecked(final boolean taskStatus, final int taskId, final String taskName) {
            Runnable runnable = () -> manager.updateTask(getApplicationContext(), taskStatus, taskId, taskName);
            runnable.run();
        }
    }
}
```

### Menu Activity

```java
public class ListMenuActivity extends AppCompatActivity {
    ListView listView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_list_menu);
        listView = (ListView) findViewById(R.id.listMenu);
        ArrayList<TodoList> lists = new ArrayList<>();
        lists.add(new TodoList("Home", "blue"));
        lists.add(new TodoList("Work", "green"));
        lists.add(new TodoList("School", "violet"));
        CustomAdapter customAdapter = new CustomAdapter(lists);
        listView.setAdapter(customAdapter);
    }

    public class CustomAdapter extends BaseAdapter {
        ArrayList<TodoList> lists;

        public CustomAdapter(ArrayList<TodoList> lists) {
            this.lists = lists;
        }

        @Override
        public int getCount() {
            return lists.size();
        }

        @Override
        public Object getItem(int i) {
            return null;
        }

        @Override
        public long getItemId(int i) {
            return i;
        }

        @Override
        public View getView(int i, View view, ViewGroup viewGroup) {
            view = getLayoutInflater().inflate(R.layout.task_menu_item, null);

            //region setting background color
            RelativeLayout layout = view.findViewById(R.id.itemBackground);
            final String color = lists.get(i).getColor();
            if (color.matches("blue"))
                layout.setBackground(ContextCompat.getDrawable(getApplicationContext(), R.drawable.list_item_background_blue));
            else if (color.matches("green"))
                layout.setBackground(ContextCompat.getDrawable(getApplicationContext(), R.drawable.list_item_background_green));
            else
                layout.setBackground(ContextCompat.getDrawable(getApplicationContext(), R.drawable.list_item_background_violet));
            //endregion

            final TextView title = view.findViewById(R.id.listName);
            title.setText(lists.get(i).getName());
            ImageButton button = view.findViewById(R.id.selectList);
            button.setOnClickListener(view1 -> listSelected(title.getText().toString(), color));

            return view;
        }

        void listSelected(String listName, String listColor) {
            Intent intent = new Intent(getApplicationContext(), ToDoListActivity.class);
            intent.putExtra("listName", listName);
            intent.putExtra("listColor", listColor);
            startActivity(intent);
        }
    }
}
```

The other resources are available in the [source code](https://github.com/nitrite/nitrite-android-demo).


