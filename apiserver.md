
# about this document 

Open to other conventions. This is what's used below.

Text surrounded in angle brackets is an example. That is, it is an
attempt at contextual explanation, not verbatim. In my verbatim
examples, I will tend to use various forms of 'service catalog' as
that was the project that kicked off this need to understand the k8s
apiserver.

# apiserver generic info

This is the conventions as I understand them. Any fault lies with my
understanding. It is likely to be self contradictory while I learn the
details of the k8s apiserver. 

Sentences surrounded with double question marks `??` indicate a lack
of understanding or questionable assertions. This allows me to write a
**statement** without qualifying it. Future versions can simply remove
the annotation or the whole statement rather than rewriting a *maybe*
or *possibly* to be a statement of truth.

Parenthetical statements are my attempt at *color commentary* of a
preceding sentence or section. Opinions and unknowns and further
questions that I need answered.

## some definitions

 - API Group - a group of related apis. A server can have one or more
   api groups. (Not sure on best-practices for splitting up between
   multiple endpoints on a single api group, vs multiple api groups.)
   Written variously as API Group apigroup api-group, etc. 

 - GroupName - related to the API Group definition. unknown restrictions on composition.

# directory structure

Some of the directory structure is required for external code
generation tooling to work. I am not sure what specific structure, but
I've been told repeatedly to follow it as k8s does it. A further
reason to follow it is I've been told to use it so external people can
come in and match it up to k8s. (Not sure this is true as there only
seem to be 1 or 2 people who understand current k8s structure).


```
  cmd/
      <binary-name>
      service-catalog
  pkg/
      apis/
           <api-group-name>
           servicecatalog/
               install/
               <versions>
               v1alpha1/
      apiserver/
      cmd/server/
      registry/
```

## cmd/

This directory contains the cli program that is run. It is mostly a
stub to set up logging and execute an spf13/cobra command.

This is where you might do some other setup.

Implicitly, this calls the `init()` functions of the apis to
install. This is done by an anonymous import of the
`apis/<apiname>/install` package of the apigroup to install. (I don't
understand this design.)

## apis/

I don't know how the naming of the subdirectories impacts the api definitions at all. 

I don't know if this is the API-Group name, or if it relates to the substructure of API-Groups at all.

?? Subdirectory naming has something to do with the code-generation. ??

I have been told not to have dashes in subdirectory names. Thus
`service-catalog` has become `servicecatalog`. ?? This is a
restriction based on code-generation. ??

## `servicecatalog/` - api definition directory

Typically three files: 
 - types.go
 - register.go
 - doc.go
Typical Subdirectories:
 - install
 - <k8s-versions>
 - v1alpha1
 - ?? validation ??

This directory will be the generic definition of an `API Group`. It
must have a base golang definition of the types to be exposed ??called
`types.go`??. ?? The definitions in this base `types.go` must be able
to hold any version of the defined resources. (Where is this
constraint defined?) ?? This is the internal-only version. (Not clear
on what internal means. Is it what storage uses?  What a controller
operating on these resources see?)

It must have an `install/` directory to do the registration of the
apigroup with the generic apiserver infrastructure. This will have an
`install.go` file, which will contain an `init()` to be imported by
the base server in `cmd/`. (This is too much spooky action at a
distance for me. I don't know the design decision that resulted in
this result. I would prefer an explicit registration/installation
call.)

There ?? must ?? be one or more versioned api directories following
the k8s versioning scheme. These tend to start off as copies of the
directory one level up, but have no install subdirectory. 

`doc.go` for code generation. ?? It contains go compiler
directives. ?? ?? It contains comments used by the code generation. ??

Generated code is contained here for ?? deepcopy ??.

### `v1alpha1` - versioned subdirectory

Contains:
 - Version specific `types.go`.
 - `register.go` for registering the types of that version,
 - `doc.go` for information about what code to generate.
 - Lots of generated code. 

### apis unknowns

not sure where conversion.go and defaults.go must be. In versioned
resources or at the toplevel unversioned resources?

## pkg/cmd/server/

The implementation of the toplevel `cmd/<server-name>/server.go` 

### Options Pattern
What I call the `Options Pattern`. A server is defined by what
`Options` structs it uses, and provides it's own `Options`. This is so
a future server can be created by importing and using it's Options
struct. 

Common `Options`:
 - Genericapiserver.ServerRunOptions
 - SecureServingOptions
 - EtcdOptions
 - DelegatingAuthenticationOptions
 - DelegatingAuthorizationOptions
 
### Cobra

All flags are cobra flags. This is done to be able to pass a cobra
struct directly to the sub-`Options`. Thus you can add configuration
capability without needing to know how to configure any of the
sub-`Options`.

## pkg/apiserver/

extension of the genericapiserver. 

Bizzare multistep process to instantiate:
 - options
 - config
 - completed
 - run

## pkg/registry/

storage information. How to store the resources defined in
apis. Stores the generic unversioned resource (which if we recall must
be able to store *any* version of a resource.

Still learning about it. ?? This exists to decouple storage from
representation. ??

# generated code

I still don't know if the generated code is **necessary** or merely
*nice to have*. Most people balk at the concept of having an apiserver
with apis and not having the generated code. I don't know if the code
**will not work** or will not work **fast**. It would be best to
leave out the generated code of an example server and worry about it
later.

 - deepcopy - ?
 - conversion - converts between different versions. for example,
   incoming v1alpha1 to internal representation, internal
   representation to outgoing v1 representation
 - defaults - ?? This is responsible for instantiating default objects such that they have some restrictions? Don't crash? ??
 - types - ?? codecgen generated optimized encode/decode for json ??
 - protobufs - ? 

?? The generated code must exist for anything to work. ?? (Why?)

# details sub directories

