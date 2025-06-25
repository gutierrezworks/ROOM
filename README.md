# ROOM

Interface for ROOM

## Features

- Room entities -> Tables
- Room DAOs -> Queries
- Database class -> Connect Database with the APP

## Getting Started

### Installation

```bash
// DataBase
// Room
- implementation("androidx.room:room-runtime:${rootProject.extra["room_version"]}")
- ksp("androidx.room:room-compiler:${rootProject.extra["room_version"]}")
- implementation("androidx.room:room-ktx:${rootProject.extra["room_version"]}")
```

### Usage

Entity:

```bash
@Entity(tableName = "items")
class Item(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val name: String,
    val price: Double,
    val quantity: Int
)
```

DAO:

```bash
@Dao
interface ItemDao {

    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun insert(item: Item)

    @Update
    suspend fun update(item: Item)

    @Delete
    suspend fun delete(item: Item)

    @Query("SELECT * FROM items WHERE id = :id")
    fun getItem(id: Int): Flow<Item>

    @Query("SELECT * FROM items ORDER BY name ASC")
    fun getAllItems(): Flow<List<Item>>
}
```

Database:

```bash
@Database(entities = [Item::class], version = 1, exportSchema = false)
abstract class InventoryDatabase: RoomDatabase() {

    abstract fun itemDao(): ItemDao

    companion object {
        @Volatile
        private var Instance: InventoryDatabase? = null
        fun getDatabase(context: Context): InventoryDatabase {
            return Instance ?: synchronized(this) {
                Room
                    .databaseBuilder(context, InventoryDatabase::class.java, "item_database")
                    .fallbackToDestructiveMigration() // HERE IS DESTROYING DATA ON MIGRATION (DON'T DO THAT)
                    .build()
                    .also { Instance = it }
            }
        }

    }

}
```

### Next steps

Create the repository:

```bash
class OfflineItemsRepository(private val itemDao: ItemDao) : ItemsRepository {
    override fun getAllItemsStream(): Flow<List<Item>> = itemDao.getAllItems()

    override fun getItemStream(id: Int): Flow<Item?> = itemDao.getItem(id)

    override suspend fun insertItem(item: Item) = itemDao.insert(item)

    override suspend fun deleteItem(item: Item) = itemDao.delete(item)

    override suspend fun updateItem(item: Item) = itemDao.update(item)
}
```

This one extends from ItemsRepository ( Interface OOP for testing and abstraction )

```bash
interface ItemsRepository {
    /**
     * Retrieve all the items from the the given data source.
     */
    fun getAllItemsStream(): Flow<List<Item>>

    /**
     * Retrieve an item from the given data source that matches with the [id].
     */
    fun getItemStream(id: Int): Flow<Item?>

    /**
     * Insert item in the data source
     */
    suspend fun insertItem(item: Item)

    /**
     * Delete item from the data source
     */
    suspend fun deleteItem(item: Item)

    /**
     * Update item in the data source
     */
    suspend fun updateItem(item: Item)
}
```

On the viewModel:

```bash
    suspend fun saveItem() {
        if (validateInput()) {
            itemsRepository.insertItem(itemUiState.itemDetails.toItem())
        }
    }
```

Repository DI:

```bash
class ItemEntryViewModel(private val itemsRepository: ItemsRepository) : ViewModel() {
```

Item STATE:

```bash
    var itemUiState by mutableStateOf(ItemUiState())
```

