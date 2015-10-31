[![CircleCI](https://circleci.com/gh/peter-edge/go-inject/tree/master.png)](https://circleci.com/gh/peter-edge/go-inject/tree/master)
[![GoDoc](http://img.shields.io/badge/GoDoc-Reference-blue.svg)](https://godoc.org/go.pedge.io/inject)
[![MIT License](http://img.shields.io/badge/License-MIT-blue.svg)](https://github.com/peter-edge/go-inject/blob/master/LICENSE)

```go
  import "go.pedge.io/inject"
```

Package inject is guice-inspired dependency injection for Go.

https://github.com/google/guice/wiki/Motivation

Concepts:

    Module

A Module is analogous to Guice's AbstractModule, used for setting up your
dependencies. This allows you to bind structs, struct pointers, interfaces, and
primitives to singletons, constructors, with or withot tags.

An interface can have a binding to another type, or to a singleton or
constructor.

    type IfaceAlpha interface {
    	Hello() string
    }

    type IfaceAlphaOne struct {
    	value string
    }

    func (i *IfaceAlphaOne) Hello() string {
    	return i.value
    }

    module := inject.NewModule()
    module.Bind((*IfaceAlpha)(nil)).To(&IfaceAlphaOne{}) // valid, but must provide a binding to *IfaceAlphaOne.
    module.Bind(&IfaceAlphaOne{}).ToSingleton(&IfaceAlphaOne{"Salutations"}) // there we go

An interface can also be bound to a singleton or constructor.

    module.Bind((*IfaceAlpha)(nil)).ToSingleton(&IfaceAlphaOne{"Salutations"})

A struct, struct pointer, or primitive must have a direct binding to a singleton
or constructor.

    Injector

An Injector is analogous to Guice's Injector, providing your dependencies.

Given the binding:

    module.Bind((*IfaceAlpha)(nil)).ToSingleton(&IfaceAlphaOne{"Salutations"})

We are able to get a value for IfaceAlpha:

    func printHello(aboveModule inject.Module) error {
    	injector, err := inject.NewInjector(aboveModule)
    	if err != nil {
    		return err
    	}
    	ifaceAlphaObj, err := injector.Get((*IfaceAlpha)(nil))
    	if err != nil {
    		return err
    	}
    	fmt.Println(ifaceAlphaObj.(IfaceAlpha).Hello()) // will print "Salutations"
    }

    Constructor

A constructor is a function that takes injected values as parameters, and
returns a value and an error.

    type IfaceBeta interface {
    	Greetings() string
    }

    type IfaceBetaOne struct {
    	ifaceAlpha IFaceAlpha
    	person string
    }

    func (i *IfaceBetaOne) Greetings() string {
    	return fmt.Sprintf("%s, %s!", i.ifaceAlpha.Hello(), i.person)
    }

We can set up a constructor to take zero values, or values that require a
binding in some module passed to NewInjector().

    func doStuff() error {
    	m1 := inject.NewModule()
    	m1.Bind((*IfaceAlpha)(nil)).ToSingleton(&IfaceAlphaOne{"Salutations"})
    	m2 := inject.NewModule()
    	m2.Bind((*IfaceBeta)(nil)).ToConstructor(newIfaceBeta)
    	injector, err := inject.NewInjector(m1, m2)
    	if err != nil {
    		return err
    	}
    	ifaceBetaObj, err := injector.Get((*IfaceBeta)(nil))
    	if err != nil {
    		return err
    	}
    	fmt.Println(ifaceBetaObj.(IfaceBeta).Greetings()) // will print "Saluatations, Alice!"
    }

    func newIfaceBetaOne(ifaceAlpha IfaceAlpha) (IfaceBeta, error) {
    	return &IfaceBetaOne{ifaceAlpha, "Alice}, nil
    }

## Usage

#### type Builder

```go
type Builder interface {
	ToSingleton(singleton interface{})
	ToConstructor(constructor interface{})
	ToSingletonConstructor(constructor interface{})
	ToTaggedConstructor(constructor interface{})
	ToTaggedSingletonConstructor(constructor interface{})
}
```

Builder is the return value from a Bind call from a Module.

#### type Injector

```go
type Injector interface {
	fmt.Stringer
	Get(from interface{}) (interface{}, error)
	GetTagged(tag string, from interface{}) (interface{}, error)
	GetTaggedBool(tag string) (bool, error)
	GetTaggedInt(tag string) (int, error)
	GetTaggedInt8(tag string) (int8, error)
	GetTaggedInt16(tag string) (int16, error)
	GetTaggedInt32(tag string) (int32, error)
	GetTaggedInt64(tag string) (int64, error)
	GetTaggedUint(tag string) (uint, error)
	GetTaggedUint8(tag string) (uint8, error)
	GetTaggedUint16(tag string) (uint16, error)
	GetTaggedUint32(tag string) (uint32, error)
	GetTaggedUint64(tag string) (uint64, error)
	GetTaggedFloat32(tag string) (float32, error)
	GetTaggedFloat64(tag string) (float64, error)
	GetTaggedComplex64(tag string) (complex64, error)
	GetTaggedComplex128(tag string) (complex128, error)
	GetTaggedString(tag string) (string, error)
	Call(function interface{}) ([]interface{}, error)
	CallTagged(taggedFunction interface{}) ([]interface{}, error)
	Populate(populateStruct interface{}) error
}
```

Injector provides your dependencies.

#### func  NewInjector

```go
func NewInjector(modules ...Module) (Injector, error)
```
NewInjector creates a new Injector for the specified Modules.

#### type InterfaceBuilder

```go
type InterfaceBuilder interface {
	Builder
	To(to interface{})
}
```

InterfaceBuilder is the return value when binding an interface from a Module.

#### type Module

```go
type Module interface {
	fmt.Stringer
	Bind(from ...interface{}) Builder
	BindTagged(tag string, from ...interface{}) Builder
	BindInterface(fromInterface ...interface{}) InterfaceBuilder
	BindTaggedInterface(tag string, fromInterface ...interface{}) InterfaceBuilder
	BindTaggedBool(tag string) Builder
	BindTaggedInt(tag string) Builder
	BindTaggedInt8(tag string) Builder
	BindTaggedInt16(tag string) Builder
	BindTaggedInt32(tag string) Builder
	BindTaggedInt64(tag string) Builder
	BindTaggedUint(tag string) Builder
	BindTaggedUint8(tag string) Builder
	BindTaggedUint16(tag string) Builder
	BindTaggedUint32(tag string) Builder
	BindTaggedUint64(tag string) Builder
	BindTaggedFloat32(tag string) Builder
	BindTaggedFloat64(tag string) Builder
	BindTaggedComplex64(tag string) Builder
	BindTaggedComplex128(tag string) Builder
	BindTaggedString(tag string) Builder
}
```

Module sets up your dependencies.

#### func  NewModule

```go
func NewModule() Module
```
NewModule creates a new Module.
