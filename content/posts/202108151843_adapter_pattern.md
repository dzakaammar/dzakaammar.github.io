---
title: "Structural Pattern: Adapter Pattern"
date: 2021-08-16T01:49:09+07:00
draft: false
type: "post"
tags: ["software_design", "design_pattern", "oop", "golang"]
---

> Bayangkan sebuah adapter dalam perangkat elektronik yang berfungsi menyambungkan antara hardware dengan sumber listrik.
> Analogi yang sama dengan *Adapter Pattern* .

Adapter pattern adalah pattern untuk menghubungkan sebuah interface yang tidak *compatible* dengan existing interface, dengan membuat sebuah interface sebagai "adapter". Secara sederhana, pattern ini digunakan untuk membuat object-object yang *incompatible* secara interface untuk saling berkolaborasi.

Adapter membungkus sebuah object untuk menyembunyikan logic dan kompleksitas *conversion*, sehingga menjadikan object tersebut menjadi *compatible*
### Karakteristik
- Implementasinya menggunakan *composition principle*, mengimplimentasi sebuah interface dan mem-*wrap* interface lainnya
- Ada sebuah *Client Interface* yang digunakan oleh *Client* sebagai adapter
- Ada sebuah *Adapter* class sebagai konkret object dari *Client Interface*, dimana logic untuk konversi ke interface yang dituju berada
- Ada sebuah *Service* a.k.a *API* a.k.a *Third Party* sebagai tujuan konversi

```go
package user

type Model struct {
	Name, Email string
}

// Repository adalah Client Interface, karena
// service yang bertindak sebagai Client tidak perlu tahu
// bagaimana atau metode apa yang digunakan.
// Secara sederhana, Service type tidak compatible dengan 
// API atau Third Party yang digunakan sebagai "repository".
type Repository interface {
	Create(u *Model) error	
}

// Service adalah client yang menggunakan Repository sebagai
// adapter untuk memenuhi logic di dalam internal proses miliknya
// sendiri
type Service struct {
	r Repository
}

func NewService(r Repository) *Service {
	return &Service{r: r}
}

func (s Service) Create(u *Model) error {
	return s.r.Create(u)
	
}

```

```go
package inmemory

// UserRepository adalah implementasi/konkret type dari Repository
// yang bertindak sebagai adapter. Dalam hal ini bertindak
// sebagai adapter ke in-memory storage (hash map)
type UserRepository struct{
	m map[string]string
}

func New() *UserRepository {
	return &UserRepository{
		m: make(map[string]string),
	}
}

func (ur *UserRepository) Create(u *user.Model) error {
	ur.m.[u.Name] = u.Email
	return nil
}

```

```go
package db

import (
	"database/sql"
	"errors"
)

// UserRepository adalah implementasi/konkret type dari Repository
// yang bertindak sebagai adapter. Dalam hal ini bertindak
// sebagai adapter ke db sql.
type UserRepository struct{
	db *sql.DB
}

func New(db *sql.DB) *UserRepository {
	return &UserRepository{
		db: db,
	}
}

func (ur *UserRepository) Create(u *user.Model) error {
	if ur.db == nil {
		return errors.New("db is nil")
	}
	return nil
}

```

```go
package main

import (
	"adapter/inmemory"
	"adapter/user"
	"adapter/db"
)

func main() {
	inmemRepo := inmemory.NewUserRepository()
	service := user.NewService(inmemRepo)
	err := service.Create(&user.Model{
		Name: "foo",
		Email: "foo@bar.com",
	})
	if err != nil { panic(err) } // tidak akan panic
	
	dbRepo := db.NewUserRepository(nil) // set nil
	service := user.NewService(dbRepo)
	err := service.Create(&user.Model{
		Name: "foo",
		Email: "foo@bar.com",
	})
	if err != nil { panic(err) } // akan panic
	
}
```

Adapter pattern akan membuat kita menyediakan *middle-layer* yang bertugas sebagai translator antara client code dengan *third-party*, *legacy class*, atau interface apapun yang tidak compatible

### Pros
- Single reponsibility principle. Kita dapat memisahkan logic konversi dengan *primary business logic*
- Open/Closed Principle. Kita dapat meng-extend atau menambah implementasi adapter baru tanpa harus merubah implementas adapter yang sudah ada.

### Cons
- 