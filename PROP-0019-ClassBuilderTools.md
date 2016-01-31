# PROP 0018 - Class Builder Tools

|                |                                              |
|:---------------|:------------------------------------------   |
| Description:   | A series of things to help build good classes|
| Status:        | In development                               |


## Purpose

I got the idea from Tiled Array to maybe have a series of base classes that things can inheret from as a
means of code factorization.  For example Tiled Array defines a base class NoDefaults, which automatically
deletes all compiler generated class members.  Such a class could be easily incorporated into an existing
class hierarchy by multiple inheritance.  Such classes are called mix-ins.  I also want to have macros for common patterns (this idea is from boost).

##Contents

My first contribution to this toolbox will be a macro for generating all the fill functions automatically.  It should be used in all container like classes.  C++11 introduces variadic templates and initializer lists, meaning for any container like class, the same fill options appear over and over again:
```C++
///Silly container like class which holds data of type T
template<typename T>
class MyClass{
private:
   //Some sort of container containing data of type T
   std::vector<T> Data_;

   tempalte<typename BeginItr_t,typename EndItr_t>
   void FillImpl(BeginItr_t& BeginItr,EndItr_t& EndItr){
     //Actually fill MyClass using the supplied iterators
   }

public:
  
  //Take a manually entered, arbitrarily long list of elements of type T
  template<typename...Args>
  void Fill(Args...args){
     std::array<T,sizeof...(Args)> Temp({args...});
     FillImpl(Temp.begin(),Temp.end());
  }
  
  //Take an initializer list of objects of type T
  void Fill(std::intializer_list<T>& IL){
    FillImpl(IL.begin(),IL.end());
  }

  //Take an arbitrary container with begin and end functions
  template<typename ConWithT>
  void Fill(ConWithT& Elems){
    FillImpl(Elems.begin(),Elem.end());
  }

  //Take iterators to the beginning and end of some container
  template<typename BeginItr,typename EndItr>
  void Fill(BeginItr& IIn,EndItr& End){
    FillImpl(Temp.begin(),Temp.end());
  }

};
```

I want to replace this with:
```C++
class MyClass{
private:
   //Some sort of container containing data of type T
   std::vector<T> Data_;

   tempalte<typename BeginItr_t,typename EndItr_t>
   void FillImpl(BeginItr_t& BeginItr,EndItr_t& EndItr){
     //Actually fill MyClass using the supplied iterators
   }

public:
  
  //Makes all of our fill fxns
  DEFINE_FILL_FXNS(T,FillImpl)


};
```
Not only is this cleaner, but it's also easily extendable.  In particular I'm thinking about fill functions that use move
semantics (which I know exist, but I haven't really looked into yet).  Thus, anyone who uses this macro would automatically get
move fill construction options, once I figure out how those work.
