.. _Introduction: 5_1_Intro_To_Module_Prog.rst
.. _Zikula: https://github.com/zikula-modules
.. _previously: 5_2_Basic_Module_Structure.rst

============================
Designing StrainID2 module
============================

:Author:
    Timothy Paustian
    
The easiest way in my opinion to undertsand how to *program* a module is to actually do it. Now that you have the basics of module programming down (you did read the previous articles right? If not start with the Introduction_), its time to work through a real world example. Everyone has their own method of programming or deciding to program. What I present here are the steps I have learned from experience and reading books on programming. If you find it useful, great, if not that's cool. Before we dive into the details, lets take a birds-eye view of the process.

- Identify and define the problem that needs to be addressed
- Determine what is required to solve the problem.
- Define, on paper, the data need to be stored, its structure and what routines you will need to access that data
- Work out the permissions and data security that you will need
- Write the Version.php file
- Write the data structure (the Entitiy classes) and initialization code Installer.php 
- Write the administration interface
- Write the user interface

Identify the Problem
====================

The first step in writing any web application is finding a problem that needs a solution. To understand where the problem that the StrainID2 module solves comes from, you need to understand a bit about what my students are doing. One of the microbiology experiments we do is an analysis of environmental samples, looking for enteric bacteria. The students isolate these microbes from the environment and then run a large number of tests to help identify them. Once identified, they are required to write a report, describing their experiment. A big problem with this process, is that there are over 120 different potential species that are commonly identified and 10 tests that need to be looked through for each. Students were spending hours paging through massive tables just to identify their isolate. A database that stored this data and a web form that was able to search through the table quickly was an excellent solution.

Find the Appropriate Solution to your Problem
=============================================

Once the problem is identified, it is now important to find the best solution. The first place to start is in available Zikula_ Github modules. You may be able to find a module that will suit your purpose and not have to program one. Many powerful and useful modules are already available and often it is easier to use a pre-made module, or modify it, than it is to start from scratch.

In the case of the project we are imagining, it is highly unlikely such a specialized module for identifying strains, would be found in at GitHub. And no surprise, there is none. So, it looks like we are going to have to roll our own.

Define the Module on Paper
==========================

A very important step is to design your module conceptually. Too often new programmers will not take the time to think through what they are trying to accomplish and may code themselves into a corner. You can save yourself days of work, by spending an hour planning out the data structures you need, the user interface, and the code to knit them together. Zikula modules are set up to encourage the use of a very powerful, and efficient design pattern, the Model-View-Controller (MVC) pattern. The model is your data stored in the database, the view is the interface that is presented to the user, typically using html pages, and the controller is the code that interfaces between the two. The files in a Zikula module as described previously_. In this phase of the project we will think about the structure of the model and how the user will view that model. 

The model for the StrainID2 module will contain the name of the strain, and then the reaction for each test. We will also add a id number and make it the primary key. This will make it easy to identify each unique entry in the table. In summary, there will be an id for each strain (integer), a name for the strain (two words, genus and species), and then the results for each test, 10 tests in all, the results of each will be represented with a + (positive), - (negative), v (varies between strains) or u (unknown). Each of these can be represented by a single character. Table 1 shows the layout of the data.

This data can naturally be stored in a single table.

+------------------+----------------------------------+-------------+--------------+
| Item Name        |   Description                    |  Type       |   Size       |
+==================+==================================+=============+==============+
|     id           |  id for the item, primary key    |   integer   |    Large     |
+------------------+----------------------------------+-------------+--------------+
|  name            |    name of the species           |   string    |     255      |
+------------------+----------------------------------+-------------+--------------+
|  indole          |    indole test                   |   string    |      1       |
+------------------+----------------------------------+-------------+--------------+
|  methyl red      |    methyl red test               |   string    |      1       |
+------------------+----------------------------------+-------------+--------------+
| vogues proskauer |    vogues proskauer test         |   string    |      1       |
+------------------+----------------------------------+-------------+--------------+
| simmon's citrate |    simmon's citrate test         |   string    |      1       |
+------------------+----------------------------------+-------------+--------------+
|    H2S           |   prod. of hydrogen sulfide      |   string    |      1       |
+------------------+----------------------------------+-------------+--------------+
| phenylalanine    |   deamination of phenyl alanine  |   string    |      1       |
+------------------+----------------------------------+-------------+--------------+
|   lysine         |  lysine decarboxylation          |   string    |      1       |
+------------------+----------------------------------+-------------+--------------+
|  ornithine       |  ornithine decarboxylation       |   string    |      1       |
+------------------+----------------------------------+-------------+--------------+
|  motility        |         motility test            |   string    |      1       |
+------------------+----------------------------------+-------------+--------------+
|  lactose         |  lactose fermentation            |   string    |      1       |
+------------------+----------------------------------+-------------+--------------+

There are three functions/interfaces that this module will need to be able to perform.

1. Display to anyone a list of strains present in the database.
2. Allow anyone to search for matching strains by giving a list of test results.
3. Allow a logged in user who is part of a registered group to enter new strains into the database

To perform these functions, several interfaces will have to be designed.

* A table listing all the strains in the database. This can be a straight html table listing each strain and its reactions
* A search interface. This will list each test and allow the user to choose the three possible reactions (+, -, u) from drop down menu lists. If you are wondering about variable (v) that is part of the data in the table, it's not a possibility. While various strains of a species can have variable reactions, they know what the reaction of *their strain* is.
* A new strain interface where an allowed user can enter the name of new strains and choose what the reactions are.
* A update strain interface where any strain information can be changed.

So there you have it, the design of our strain module. Now its time to get coding.