# In-memory Repository in Go

Below is the example of in-memory implementation of Repository pattern in Go. 

```go
package main

import (
	"context"
	"crypto/rand"
	"errors"
	"fmt"
)

// List of error values.
var (
	ErrUserNotFound = errors.New("user is not found")
)

// User represents the user that uses the application.
type User struct {
	ID             string
	Name           string
	Phone          string
	Email          string
	PictureURL     string
	HashedPassword string
}

// UserRepository implements operations for working with the user data storage.
type UserRepository struct {
	data []*User
}

// FindByID finds an user by the id.
func (r *UserRepository) FindByID(ctx context.Context, userID string) (*User, error) {
	for _, d := range r.data {
		if d.ID == userID {
			return d, nil
		}
	}
	return nil, ErrUserNotFound
}

// FindByEmail finds an user by the email.
func (r *UserRepository) FindByEmail(ctx context.Context, email string) (*User, error) {
	for _, d := range r.data {
		if d.Email == email {
			return d, nil
		}
	}
	return nil, ErrUserNotFound
}

// Update updates an user in the data storage.
func (r *UserRepository) Update(ctx context.Context, usr *User) error {
	for _, d := range r.data {
		if d.ID == usr.ID {
			d = usr
			return nil
		}
	}
	return nil
}

// Insert inserts an user to the data storage.
func (r *UserRepository) Insert(ctx context.Context, usr *User) error {
	usr.ID = randomHex(32)
	r.data = append(r.data, usr)
	return nil
}

// Generate a random hex.
func randomHex(len int) string {
	buf := make([]byte, len)
	_, err := rand.Read(buf)
	if err != nil {
		panic(err)
	}
	return fmt.Sprintf("%x", buf)
}

// Delete deletes an user.
func (r *UserRepository) Delete(ctx context.Context, userID string) error {
	for i, d := range r.data {
		if d.ID == userID {
			r.data = append(r.data[:i], r.data[i+1:]...)
			return nil
		}
	}
	return nil
}

func main() {
	repo := &UserRepository{
		data: []*User{},
	}

	usr := &User{
		Name:           "Alice",
		Phone:          "0123456789",
		Email:          "alice@wonderland.com",
		PictureURL:     "https://en.wikipedia.org/wiki/File:Alice_in_wonderland_1951.jpg",
		HashedPassword: "password",
	}

	// insert an suser
	_ = repo.Insert(context.Background(), usr)

	// find alice
	alice, _ := repo.FindByID(context.Background(), usr.ID)
	fmt.Printf("alice: %#v\n", alice)

	// update alice
	alice.Name = "Bob"
	_ = repo.Update(context.Background(), alice)
	fmt.Printf("new alice: %#v\n", alice)
}
```

