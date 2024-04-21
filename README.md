# GORM libSQL Driver

An unofficial driver for using libSQL/Turso with Gorm.

This is a fork of the [official Gorm SQLite](https://github.com/go-gorm/sqlite) driver changed to work with libSQL/Turso. The primary change is the introduction of a Config struct.

Note: This driver does not contain a default SQLite driver like the original does. This allows you to import your choice of the Turso/libSQL Go client and pick the features that are important for you.

## Full feature usage

This version allows you to use all of the features of Turso/libSQL, including embedded replicas, but requires cGo to be enabled to work as expected.

```go
import (
	"database/sql"
	"fmt"
	"log"
	"os"
	"path/filepath"
	"time"

	"github.com/tursodatabase/go-libsql"
  sqlite "<path to this package>"
	"gorm.io/gorm"
)

// Create your Turso/libSQL connection
dbName := "local.db"
primaryUrl := os.Getenv("DB_URL")
authToken := os.Getenv("DB_AUTH_TOKEN")
dbPath := filepath.Join("./db", dbName)
syncInterval := time.Minute

connector, err := libsql.NewEmbeddedReplicaConnector(dbPath, primaryUrl,
	libsql.WithAuthToken(authToken),
	libsql.WithSyncInterval(syncInterval),
)
if err != nil {
	fmt.Println("Error creating connector:", err)
	os.Exit(1)
}

// Pass a database connection to Gorm using this driver
conn := sql.OpenDB(connector)
db, err := gorm.Open(sqlite.New(sqlite.Config{
	Conn: conn,
}), &gorm.Config{})
if err != nil {
	log.Fatalf("Error connecting to database: %v", err)
}

```

## Pure go usage

For those willing to not use embedded replicas, you will not need cGo enabled. Turso/libSQL offers a pure Go driver that we can make use of.

The libSQL client must be imported as an unnamed import so we can make use of it's side effects (acting as the driver for Gorm)

```go
import (
	"database/sql/driver"
	"errors"
	"fmt"
	"os"

	_ "github.com/tursodatabase/libsql-client-go/libsql"
	"gorm.io/gorm"
	sqlite "<path to this package>"
)

dburl := os.Getenv("DATABASE_URL")
var err error

// Pass a DSN or Turso DB url to Gorm. The url must also include an authToken as a query parameter
conn, err := gorm.Open(sqlite.New(sqlite.Config{
	DSN:        dburl,
}), &gorm.Config{})

if err != nil {
	fmt.Println("Failed to connect to database")
	panic(err)
}
```

## Useful links

- [Gorm docs](https://gorm.io/)
- [Gorm SQLite driver docs](https://github.com/go-gorm/sqlite)
- [Turso Go SDK docs](https://docs.turso.tech/sdk/go/quickstart)
