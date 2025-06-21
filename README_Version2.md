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
                    .fallbackToDestructiveMigration()
                    .build()
                    .also { Instance = it }
            }
        }

    }

}
```