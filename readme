

# Room Database Guide Coroutine ( Kotlin )
Add in build.gradle(Module:app)  
Notes by Rhytham Negi
## Installation
 
```bash
 plugins {
    .....
    id 'kotlin-kapt'
    // To use Kotlin annotation processing tool (kapt)
}

dependencies {
    def room_version = "2.5.1"

    implementation "androidx.room:room-ktx:$room_version"
    // To use Kotlin annotation processing tool (kapt)
    kapt "androidx.room:room-compiler:$room_version"
}
```
    
## 
While working with Database, specially with Room Database we need 3 thing :
### 1. Entity in a Database:
    1.Tables (Database table)
    2.Attributes (name, address, phone number,..etc [ Data inside one Row ])
    3.Fields ( Data inside each column.)
 Example:-
 - name : "Rhytham Negi",
 - phoneno : 1234567890,
 - address: "Amrika Colony"
 ### 2. Database :
 Where all the table or Entity of Room Database stored.

 ### 3. Specially for Room DAO:
We need DAO“Data Access Object” interface with describe how we interact with Database.
 



## Let's Start
### Step 1:

Create a new data Class that define our table in the database.
Example Code:
```
// Use @Entity annotation in Data class to create table in Room Database Table

@Entity
data class Contact(
    val firstName: String,
    val lastName: String,
    val phoneNumber: String,

// Use @PrimaryKey annotation to make "id" as Primary Key for Table

    @PrimaryKey(autoGenerate = true)
    val id: Int = 0

)

```
### Step 2:

In the next step, create an interface DAO for our Entity tabe. That's stores all the operations or functions we want to perform.
Example Code:
```
// Assign this interface as DAO for our Room Database.

@Dao
interface ContactDao{

// Due to the kotlin annotation, For some operations not to write Query for that.

 @Upsert
    suspend fun upsertContact(contact: Contact)

    @Delete
    suspend fun deleteContact(contact: Contact)

    @Query("SELECT * FROM contact ORDER BY firstName ASC")
    fun getContactsOrderedByFirstName(): Flow<List<Contact>>

    @Query("SELECT * FROM contact ORDER BY lastName ASC")
    fun getContactsOrderedByLastName(): Flow<List<Contact>>

    @Query("SELECT * FROM contact ORDER BY phoneNumber ASC")
    fun getContactsOrderedByPhoneNumber(): Flow<List<Contact>>
}
```

#### Flow<>
Flow<> is use to notify any changes in database 

#### suspend
suspend function runs in co-routine and block until the database operation is closed.

#### @Upsert and @Delete
We use DAO annotations where SQL query not required.

#### @Query
Use to write SQL query.

### Step 3:
Now, Final step is to tell the Room Database about DAOs and Entities.

Create an abstract class that inherit the Room Database.

Code Example:
```
// In @Database define all the entities
@Database(
    entities=[Contact::class],
    version=1
)
abstract class ContactDatabase: RoomDatabase(){
    
    //then, we need to define abstract val for our DAOs.
    abstract val dao: ContactDao
}
```
### Step 4
Now, create a sealed interface that defines all the Events in our Compose UI. Like "Click on floating button", "delete button click", ... etc.

Code Example:
```
sealed interface ContactEvent{
    object SaveContact: ContactEvent,
    data class SetFirstName(val firstName:String):ContactEvent,
    data class SetLastName(val lastName:String):ContactEvent,
    data class SetPhoneNumber(val phoneNumber:String):ContactEvent,
    object ShowDialog:ContactEvent,
    object HideDialog:ContactEvent,
    data class DeleteContact(contact:Contact):ContactEvent,

    // For Sorting Events, We need to create Enum that contains all the objects or Events for SortType the data.

    data SortContacts(sortType:SortType):ContactEvent
}
```
#### Enum
For Contact Sort Type (Example : sortByName,sortByPhoneNumber,... etc) 
Create an Enum ```SortType```

Code Example:
```
enum class SortType{
    FIRST_NAME,
    SECOND_NAME,
    PHONE_NUMBER
}
```
For the UI perspective, need to create a data class that stores all the states of our UI.
Example : isContactAdded, 
[Default value for] firstName,... etc.

Code Example:
```
data class ContactState{
    val contacts:List<String> = emptyList(),
    val firstName:String = "",
    val lastName:String = "",
    val phoneNumber:String = "",
    val isAddingContact:Boolean = false,
    val sortType:SortType = SortType.FIRST_NAME
}
```
### Step 5
Now, we need to create View Model class.
The ViewModel is responsible to provide data to the UI and survive configuration changes such as screen rotations. 

Create a kotlin class ContactViewModel and extend with ViewModel().
```
class ContactViewModel(
    private val dao:ContactDao
) : ViewModel(){

    private _sortType = MutableStateFlow(SortType.FIRST_NAME)
    private val _contacts = _sortType
        .flatMapLatest { sortType ->
            when(sortType) {
                SortType.FIRST_NAME -> dao.getContactsOrderedByFirstName()
                SortType.LAST_NAME -> dao.getContactsOrderedByLastName()
                SortType.PHONE_NUMBER -> dao.getContactsOrderedByPhoneNumber()
            }
        }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(), emptyList())

    private val _state = MutableStateFlow(ContactState())
    val state = combine(_state, _sortType, _contacts) { state, sortType, contacts ->
        state.copy(
            contacts = contacts,
            sortType = sortType
        )
    }.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), ContactState())
    private _state = MutableStateFlow(ContactState())
    /*
     MutableStateFlow to represent the state of a view and update  it based on user input or other events
    */

    fun onEvent(event:ContactEvent){
          when(event) {
            is ContactEvent.DeleteContact -> {
                viewModelScope.launch {
                    dao.deleteContact(event.contact)
                }
            }
            ContactEvent.HideDialog -> {
                _state.update { it.copy(
                    isAddingContact = false
                ) }
            }
            ContactEvent.SaveContact -> {
                val firstName = state.value.firstName
                val lastName = state.value.lastName
                val phoneNumber = state.value.phoneNumber

                if(firstName.isBlank() || lastName.isBlank() || phoneNumber.isBlank()) {
                    return
                }

                val contact = Contact(
                    firstName = firstName,
                    lastName = lastName,
                    phoneNumber = phoneNumber
                )
                viewModelScope.launch {
                    dao.upsertContact(contact)
                }
                _state.update { it.copy(
                    isAddingContact = false,
                    firstName = "",
                    lastName = "",
                    phoneNumber = ""
                ) }
            }
            is ContactEvent.SetFirstName -> {
                _state.update { it.copy(
                    firstName = event.firstName
                ) }
            }
            is ContactEvent.SetLastName -> {
                _state.update { it.copy(
                    lastName = event.lastName
                ) }
            }
            is ContactEvent.SetPhoneNumber -> {
                _state.update { it.copy(
                    phoneNumber = event.phoneNumber
                ) }
            }
            ContactEvent.ShowDialog -> {
                _state.update { it.copy(
                    isAddingContact = true
                ) }
            }
            is ContactEvent.SortContacts -> {
                _sortType.value = event.sortType
            }
        }
    }


}


```
```ContactViewModel``` which is used to manage the UI state and data associated with displaying and manipulating a list of contacts.

- ```_sortType```: This is a private property that is an instance of ```MutableStateFlow``` class, which is part of the Kotlin coroutines library. It represents the current sort order for the list of contacts (by first name, last name, or phone number). It is initialized with a default value of SortType.FIRST_NAME.

- ```_contacts```: This is a private property that is a flow of the list of contacts. It is generated from the``` _sortType``` property using the ```flatMapLatest``` operator, which allows us to switch between different flow sources depending on the ```_sortType``` value. Whenever ```_sortType``` changes, this flow will emit a new list of contacts sorted according to the updated _```sortType```. In other words, ```_contacts``` is a ```Flow<List<Contact>>``` that emits a list of contacts based on the current sort type selected by the user. The sorting is determined by the ```SortType``` ```enum```, which can be sorted by ```firstName```, ```lastName``` or ```phoneNumber```. The ```flatMapLatest``` operator is used to switch between different Flows based on the latest value emitted by ```_sortType```. Depending on the sort type, it will return a different Flow of contacts from the database through the Data Access Object (DAO) associated with this ```ViewModel```. The resulting Flow of contacts is then transformed into a ```List<Contact>``` using the ```stateIn`` operator. This means that the ```List<Contact>``` will only be recomputed when there are new subscribers, and it will start emitting immediately upon subscription. The initial value is an empty list.

- ```_state```: This is a private property that holds the current state of the UI-related data: whether we are adding/editing/deleting a contact, the values of the text fields etc. It is an instance of ```MutableStateFlow<ContactState>```.

- ```state```: This is a public property that is derived from the ```_state```, ```_sortType```, and ```_contacts``` properties using the combine operator. It emits a single object of type ```ContactState``` which contains all the state information required to render the UI. It is an instance of ```StateFlow<ContactState>```.

The onEvent method, which handles incoming events triggered by user interactions:

- We use a ```when``` expression to handle different types of events as sealed classes (a type-safe way of defining restricted hierarchies of classes).

- If the event is to delete a contact, then a coroutine is launched to delete the contact from the database via the ```dao``` object.

- If the event is to hide a dialog (e.g., when user clicks cancel), then we update the ```_state``` property to indicate that we are no longer adding a contact.

- If the ```event``` is to save a new contact, we extract the ```firstName```, ```lastName```, and ```phoneNumber``` from the current state object. We then check if any of these fields are empty, and if so, we return without saving the contact. Otherwise, we create a new contact object and launch a coroutine to insert it into the database via the dao object. Finally, we update the ```_state``` object to indicate that we are no longer adding a contact, and clear the text fields.

- If the ```event``` is to set the value of a text field (e.g., when the user types in their name or phone number), then we update the corresponding value in the ```_state``` property.

- If the ```event``` is to show the add contact dialog, then we update the ```_state``` property to indicate that we are now adding a contact.

- If the ```event``` is to sort the contacts, then we update the ```_sortType``` property to change the order in which the contacts are sorted.

# Compose App UI

Now, We create our UI Files Based on Jetpack Compose with implementations:
1. Contact screen
2. Add Contact dialog.

##### Create ```ContactScreen``` File.

Code Example:
```
@Composable
fun ContactScreen(
    state: ContactState,
    onEvent: (ContactEvent) -> Unit
) {
    Scaffold(
        floatingActionButton = {
            FloatingActionButton(onClick = {
                onEvent(ContactEvent.ShowDialog)
            }) {
                Icon(
                    imageVector = Icons.Default.Add,
                    contentDescription = "Add contact"
                )
            }
        },
    ) { _ ->
        if(state.isAddingContact) {
            AddContactDialog(state = state, onEvent = onEvent)
        }

        LazyColumn(
            contentPadding = PaddingValues(16.dp),
            modifier = Modifier.fillMaxSize(),
            verticalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            item {
                Row(
                    modifier = Modifier
                        .fillMaxWidth()
                        .horizontalScroll(rememberScrollState()),
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    SortType.values().forEach { sortType ->
                        Row(
                            modifier = Modifier
                                .clickable {
                                    onEvent(ContactEvent.SortContacts(sortType))
                                },
                            verticalAlignment = CenterVertically
                        ) {
                            RadioButton(
                                selected = state.sortType == sortType,
                                onClick = {
                                    onEvent(ContactEvent.SortContacts(sortType))
                                }
                            )
                            Text(text = sortType.name)
                        }
                    }
                }
            }
            items(state.contacts) { contact ->
                Row(
                    modifier = Modifier.fillMaxWidth()
                ) {
                    Column(
                        modifier = Modifier.weight(1f)
                    ) {
                        Text(
                            text = "${contact.firstName} ${contact.lastName}",
                            fontSize = 20.sp
                        )
                        Text(text = contact.phoneNumber, fontSize = 12.sp)
                    }
                    IconButton(onClick = {
                        onEvent(ContactEvent.DeleteContact(contact))
                    }) {
                        Icon(
                            imageVector = Icons.Default.Delete,
                            contentDescription = "Delete contact"
                        )
                    }
                }
            }
        }
    }
}
```

##### Create ```AddontactDialog``` File.

Code Example:
```
@Composable
fun AddContactDialog(
    state: ContactState,
    onEvent: (ContactEvent) -> Unit,
    modifier: Modifier = Modifier
) {
    AlertDialog(
        modifier = modifier,
        onDismissRequest = {
            onEvent(ContactEvent.HideDialog)
        },
        title = { Text(text = "Add contact") },
        text = {
            Column(
                verticalArrangement = Arrangement.spacedBy(8.dp)
            ) {
                TextField(
                    value = state.firstName,
                    onValueChange = {
                        onEvent(ContactEvent.SetFirstName(it))
                    },
                    placeholder = {
                        Text(text = "First name")
                    }
                )
                TextField(
                    value = state.lastName,
                    onValueChange = {
                        onEvent(ContactEvent.SetLastName(it))
                    },
                    placeholder = {
                        Text(text = "Last name")
                    }
                )
                TextField(
                    value = state.phoneNumber,
                    onValueChange = {
                        onEvent(ContactEvent.SetPhoneNumber(it))
                    },
                    placeholder = {
                        Text(text = "Phone number")
                    }
                )
            }
        },
        buttons = {
            Box(
                modifier = Modifier.fillMaxWidth(),
                contentAlignment = Alignment.CenterEnd
            ) {
                Button(onClick = {
                    onEvent(ContactEvent.SaveContact)
                }) {
                    Text(text = "Save")
                }
            }
        }
    )
}
```

##### MainActivity (To Wrap up the Code).

Code Example:
```
class MainActivity : ComponentActivity() {

    private val db by lazy {
        Room.databaseBuilder(
            applicationContext,
            ContactDatabase::class.java,
            "contacts.db"
        ).build()
    }
    private val viewModel by viewModels<ContactViewModel>(
        factoryProducer = {
            object : ViewModelProvider.Factory {
                override fun <T : ViewModel?> create(modelClass: Class<T>): T {
                    return ContactViewModel(db.dao) as T
                }
            }
        }
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            RoomGuideAndroidTheme {
                val state by viewModel.state.collectAsState()
                ContactScreen(state = state, onEvent = viewModel::onEvent)
            }
        }
    }
}
```
##  </Codie> End

In this guide, we have learned how to use Room Database with Coroutine in Kotlin to create a simple app that stores and displays user data. We have seen how to create entities, DAOs, and databases using Room annotations and extensions. We have also learned how to perform CRUD operations using suspend functions and Flows. Finally, we have explored how to use viewModelScope to run database operations on the background thread and update the UI on the main thread. Room Database with Coroutine in Kotlin is a powerful and convenient way to work with local data in Android apps.

