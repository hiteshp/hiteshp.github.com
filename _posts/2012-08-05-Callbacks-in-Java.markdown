---
comments: true
layout: post
title: Callbacks in Java
category: Coding
tags: callbacks java
year: 2012
month: 8
day: 5
published: true
summary: Implementing callbacks in Java using Interfaces
---

<h5>Goal</h5>
Work with any type of data.   
We can use callbacks for this purpose. In Java callbacks can be implemented using interfaces.

<h5>Callbacks</h5>
Client passes object to work routine.   
The routine callback Objects function to perform appropriate actions for that type as needed.   

<!-- more start -->

* Client
* routine
* Object
* Interface   
  _(behind Object)_


The interface provides common functions for all Object types.

    public interface Portable {
         public void selectResource();
    }

Lets create few Object types:

    public class Mobile implements Portable {
        public String type;

        public Mobile(String type){
            this.type = type;
        }

        public void selectResource(){
            System.out.println("Device: "+ type);
            System.out.println(">Selecting mobile image");
        }
    }

And one more:

    public class Tablet implements Portable {
        public String type;
      
        public Tablet(String type){
            this.type = type;
        }

        public void selectResource(){
            System.out.println("Device: "+ type);
            System.out.println(">Selecting Hi-Res image for Tablet");

        }
    }

The routine method implementation will be oblivious to type of device:

    public class App {
        public void displayResource(Portable type){
            type.selectResource();
        }
    }

Finally, program that uses routine:

    public class InstalledClient {
        public static void main(String[] args){

            App instance1 = new App();
            Portable type1 = new Mobile("Wildfire");

            instance1.displayResource(type1);

 

            //test for another type
            
            System.out.println("Another device...");
            type1 = new Tablet("Nexus 7");

            instance1.displayResource(type1);

        } 
    }

The client calls displayResource() routine, which does not know type of device.   
displayResource() call back Objects method to perform right function.

<!-- more end -->
