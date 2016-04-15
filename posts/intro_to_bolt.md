---
template: post
title: Intro to Bolt and key/value stores
sitename: Blog Blair
---

###  Introduction to Bolt and key/value stores
While exploring Go I was introduced to key/value stores via
[Bolt](https://github.com/boltdb/bolt). I hadn't heard of Redis or the
[Little Redis Book](http://openmymind.net/redis.pdf) which would have been a
helpful introduction to key/value stores.

When I was learning about Bolt I really wanted a tangible example. So the goal
behind this post was to create a short simple overview of key/value stores and
how to use them in the context of Bolt.

##### Objectives
1. Introduce key/value stores
2. Have a live example of Bolt working
3. Code some Go!

If you want to immediately get ankle deep in an implemented Bolt store click
[here](https://github.com/amaxwellblair/crud_app/blob/master/app/models/robotStore.go)

##### Getting started
Install [Go](https://golang.org/doc/install) if you have not

Basic understanding of programming and Go

Basic understanding of databases

It would also be helpful to have some knowledge of HTTP

#### What is Bolt?
The repository README for Bolt is informative, concise and clear. Take a read
through if you like but I will quote the README here:

>Bolt is a pure Go key/value store inspired by Howard Chu's LMDB project. The
goal of the project is to provide a simple, fast, and reliable database for
projects that don't require a full database server such as Postgres or MySQL.
>Since Bolt is meant to be used as such a low-level piece of functionality,
simplicity is key. The API will be small and only focus on getting values and
setting values. That's it.

#### What is a key/value store?
If you have used a modern programming language you have probably interacted with
 a hash map. Here are some examples of hash map syntax:

1. Ruby: `hash`
2. Go: `map`
3. Python: `dictionary`

Using a hash map is along the same lines as using a key/value store.

A key/value store is simply a data structure that uses specific keys
to index values. If we were to give our store a key it should return the value
associated with it.

#### So why use Bolt?
So why should we even use Bolt if most languages have a key/value store
standard?

1. Persistent: Since a server is stateless we will need to write these key/value
pairs somewhere. Bolt can write these directly to a file in memory. No set up
process or server needed to get started.
2. Thread management: Will lock your key/value store automatically if multiple
threads are trying to access write access.
3. Independent: Written in Go for Go.
4. Mobile: iOS and Android support
5. Simple: Enough said

#### How to use Bolt
The remainder of this introduction will go through creating a model for a basic
CRUD app using Bolt and TDD.

*Note: I will be excluding imports from this post. Please check out
[goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) or
[go-plus](https://atom.io/packages/go-plus) (if you are an atom user)*

To begin we will write a test to create a robot entry and retrieve it from the
database.

```
package robots

// Testing functionality for robot read and create methods
func TestStore_CreateRobot_CreateandRetrieveARobot(t *testing.T) {
	s := MustOpenStore()
	defer Close(s)
	name := "Roboto"
	function := "Bend things that are hard to bend"

	s.CreateRobot(name, function)
	all, err := s.All()
	if err != nil {
		t.Fatal(err)
	}

	if all[len(all)-1].Name != "Roboto" {
		t.Fatalf("unexpected name %s", all[len(all)-1].Name)
	}
}
```

OK so none of this stuff works. Let's break it down piece by piece.

`MustOpenStore()` let's first build a function to open up our Bolt store.

```
// Store will hold the database
type Store struct {
	path string
	db   *bolt.DB
}

// NewStore returns a database
func NewStore(path string) *Store {
	return &Store{
		path: path,
	}
}

// Open creates or opens a the database at the given path
func (s *Store) Open() (err error) {
	s.db, err = bolt.Open(s.path, 0600, &bolt.Options{Timeout: 1 * time.Second})
	if err != nil {
		return err
	}

	if err := s.db.Update(func(tx *bolt.Tx) error {
		tx.CreateBucketIfNotExists([]byte("robots"))
		return nil
	}); err != nil {
		return err
	}
	return nil
}

// Close closes the open database
func (s *Store) Close() {
	s.db.Close()
}
```

I will skip over some of the more generic pieces and dive right into the Bolt.
`bolt.Open` will create or find a database at the given path (first argument).
The `0600` is a field for the file type which can be changed given your
preferred permissions. The final argument which includes `Timeout` can be found
verbatim in the README:
>Please note that Bolt obtains a file lock on the data file so multiple
processes cannot open the same database at the same time. Opening an already
open Bolt database will cause it to hang until the other process closes it. To
prevent an indefinite wait you can pass a timeout option

I'd prefer not to have my server hang so I have placed in the recommended
timeout.

We also create our first bucket within our `Open()` function. When ever
writing to our database we need to use the Bolt function `Update`. This takes in
 a function where a transaction occurs.

In this specific function we are using `CreateBucketIfNotExists` which takes in
a ID in bytes. Our bucket will hold our key/value pairs or even more nested
buckets.

We have also included a `Close()` function which should be run as a `defer`
whenever we `Open()` the database

OK - now with a working open/close we can create our `MustOpenStore()`. This
function is specifically for testing the functionality of our model.

```
func MustOpenStore() *robots.Store {
	s := robots.NewStore("../../db/test.db")
	s.Open()
	return s
}

func Close(s *robots.Store) {
	s.Close()
	os.Remove("../../db/test.db")
}
```

With these functions in place we can now properly set up and teardown our tests
when proceeding with our TDD.
