# Ohm Cheatsheet

** Ohm **: a library for storing data in Redis, a key-value store

** Ohm Version **: 2.3.0

## I. Prerequisites

* Redis
* Ruby

## II. Installation
Just install the `ohm` gem, fire up an irb session or write a Ruby script and require it.

    gem install ohm

## III. Connecting to Redis
#### A. Single Connection

By default, Ohm connects to Redis at localhost (127.0.0.1), port 6379. If you wish to override this, you may set a different Redis URL with Redic, a lightweight Redis client.

    require "ohm"
    Ohm.redis = Redic.new("redis://<IP>:<PORT>")
    
    # Sample: Ohm.redis = Redic.new("redis://58.163.21.31:6379")

#### B. Multiple Connections

Certain Models could connect to different Redis servers.

    Ohm.redis = Redic.new(REDIS_URL1)

    class Student < Ohm::Model
    end

    Student = Redic.new(REDIS_URL2)

## IV. Simple Key-Value Fetch and Get

    require "ohm"

    Ohm.redis.call "SET", "Key", "Value"

    Ohm.redis.call "GET", "Key"
    # => "Value"

## V. Mapping Objects to Key Value Store

### A. Class Declaration

    class Student < Ohm::Model
    end

### B. Classes Attributes, References, and More
     class Student < Ohm::Model
       attribute :student_number
       attribute :first_name
       attribute :last_name
       attribute :birthdate
       
       index :student_number
     end

    class Course < Ohm::Model
      attribute :name
      attribute :domain
      reference :classroom, :Classroom
      set :students, :Student
      counter :waitlist
      counter :drops
      
      index :name
    end

    class Classroom < Ohm::Model
      attribute :name
      collection :courses, :Course
    end

#### i. attribute

Any value that can be stored in a string. Numbers stored would be returned as a string.

#### ii. set
Similar to an unordered list.

##### a. Adding To a Set

    course = Course.create(name: 'English 1')

    student1 = Student.create(first_name: 'Adelen', last_name: 'Festin')
    course.students.add(student1)

    course.students.add(Student.create(first_name: 'Victoria', last_name: 'Po'))
    
    course.students.count
    # => 2



##### b. Iterating From a Set

Elements of a set are returned like enumerables to which you can apply iterative methods.

    course.students.each do |student|
    puts student.last_name
    end
    # => Festin
    #    Po

    course.students.map(&:first_name)
    # => ["Adelen", "Victoria"]

##### c. Removing From a Set
  
* index starts at 1

** Proper way: **

Usage of the delete method on the set to actually remove the element from the set.

    course.students.delete(course.students[2])


** Improper way: **

Deleting the object itself.

    student = course.students[1]
    student.delete
    
Doing so would result to having the deleted element still part of the list

    course.students.count
    # count would still include deleted element

But accessing the set element would result to nil:

    course.students[1]
   
    course.students.map(&:id)
    # [.., nil, ..]
  

#### iii. list

    class PreenlistmentList
    reference :course, :Course
    list :students, :Student
    end
    
    taekwondo = Course.create(name: 'PE 2 TKD')
    
    student1 = Student.create(student_number: '2010-00033')
    student2 = Student.create(student_number: '2010-18415')
    student3 = Student.create(student_number: '2010-30011')
    
    pl = PreenlistmentList.create(course: taekwondo)
    
    # Pushes element to end of list
    pl.students.push(student1)
    pl.save # necessary for list to persist
    pl.students.size
    
    # Places element to front of list
    pl.students.unshift(student2)
    pl.save
    pl.students.size
    
    
    # Push student3 twice
    pl.students.push(student3)
    pl.save
    pl.students.size
    # => 3
    
    pl.students.push(student3)
    pl.save
    pl.students.size
    # => 4
    
    # Deletes all occurences of student3
    pl.students.delete(student3)
    
    pl.students.size
    # 2
    


#### iv. counter

Just like a regular attribute but direct manipulation and assignment is not allowed. Can only increment or decrement.

    Course[1].incr(:waitlist) # 0 + 1
    # => 1

    course = Course[1]
    course.incr(:waitlist) # 1 + 1
    course.waitlist
    # => 2
    
    course.decr(:waitlist) # 2 - 1
    # => 1
 
For multiple attributes, you may all increase and/or decrease in one line by separating the attributes with a comma.

    course.decr(:waitlist, :drops)
    

#### v. reference

Reference to another model; similar to a foreign key.

    ph108 = Classroom.create(name: 'Palma Hall, Room 108')
    
    course = Course.create(name: 'Geog 1', classroom: ph108)
    
    course.classroom
    # <Classroom:0x007f8dc9f2bb70 @attributes= ...
    
    course.classroom.name
    # "Palma Hall, Room 108"


#### vi. collection

A shortcut accessor to search for all models that reference the current model. Returns an enumerable

  Classroom[1].courses
  
  Classroom[1].courses.count


## VI. CRUD Operations

#### A. Creating Records

##### i. Immediate Create
    course = Course.create name: 'Math 17'
    
    course.id
    # => "1"
    
    course.name
    # => "Math 17"
    
    Course[1]
    
##### ii. Initialize and Save

  another_course = Course.new name: 'CS 11'
  
  another_course.name
  # => "CS 11"
  
  another_course.save

#### B. Reading / Looking up a record

##### i. By Index

    course = Course[1]
    course.name
    # => "Math 17"
    
##### ii. By Query (id)
    course = Course.find(id: 1).first
    course.name
    # => "Math 17"

##### iii. By Query (attributes)
    course = Course.find(name: 'Math 17').first
    course.name
    # => "Math 17"


#### C. Updating Records
    course = Course.find(name: 'Math 17').first
    course.id
    # => 1
    
    course.update(name: "Math 53")
    course.name
    # => "Math 53"
    
    course.id
    # => 1


#### D.Deleting Records

##### i. Direct Access
  
    Course[1].delete

    
##### ii. Prior Assignment
    
    course = Course[1]
    course.delete


## VII. Filtering

#### A. Single Attribute

    Course.find(domain: 'Math')

#### B. Multiple Attribute

    Course.find(domain: 'English', waitlist: 0)

#### C. Multiple Values for an Attribute

  # Find all courses with waitlist count = 0 and
  # has domains of either Math or English
  
    Course.find(waitlist: 0).combine(domain: ["Math", "English"])

#### D. With Exceptions
  # Find all courses under the Math domain except
  # for courses with Math 2 as the name
    
    Course.find(domain: "Math").except(name: "Math 2")


## VIII. Indices

Adding indices to models would allow you to execute find operations on the indexed attributes.

    class People < Ohm::Model
    attribute :name
    attribute :gender
  
    index :name

    end

** Valid find lookups **:

    People.find(name: 'John Doe')

** Invalid find lookups **:

  People.find(gender: 'Female')
  # => Ohm::IndexNotFound: Ohm::IndexNotFound
  
  # To fix: add gender to the index list
  # index :name, :gender


## IX. Sorting

All Sets can be sorted with `sort` which by default sorts by ID but can be overriden when passed with the `by` parameter.

*On Ohm Version 2.3.0, you can only sort by numeric fields else a runtime error or an unsorted output may result*


#### A. By

Indicate the attribute to which sorting will be based on.

For **accuracy**, use sort_by

     courses = Course.all.sort_by(:waitlist)

     courses.map(&:waitlist)
     => [0, 0, 1]
     

##### B. Order

    Course.all.sort_by(:waitlist, order: 'ASC').map(&:waitlist)
    # => [0, 0, 1]

    Course.all.sort_by(:waitlist, order: 'DESC').map(&:waitlist)
    # => [1, 0, 0]


#### B. Limit

    Course.all.sort(limit: [1, 2])
    # Gets 2 entries starting from offset 1

    Course.all.sort(limit: [0, 1])
    # Gets the first entry


## X. Uniqueness

    class Room < Ohm::Model
      attribute :name
      attribute :building
      unique :name
    end
    
    Room.create(name: 'Rm 180', building: 'Palma Hall')
    # success
    
    Room.create(name: 'Rm 180', building: 'DCS')
    # Ohm::UniqueIndexViolation: UniqueIndexViolation: name

