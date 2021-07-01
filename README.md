# OWLS-FDplan

## About
OWLS-FDplan is a service composition planner: Given a service composition problem expressed in OWL-S 1.1 as input, OWLS-FDplan produces a set of plans solving that problem. This document gives users a complete example to use the OWLS-FDplan. 

It consists of two steps: 1) using OWLS2PDDL to convert OWL files into PDDL files; 2) invoking FastDownwardCaller on this PDDL representation to produce the set of plans (OWL-S).

## Authors
This software is originally adapted and developed by Anthony Heggen, and currently maintained by Guangyi Chen.
Contact: Dr.Elena Jaramillo, [PD Dr. Matthias Klusch](https://www.dfki.de/~klusch/)

## To run
### 1 OWL2SPDDL
#### 1.1 Local File System
The current version of OWLS2PDDL requires read/write access to a local directory. This directory must be hosted at http://127.0.0.1:8080, and should have the following structure:

```
root folder
  services
    <all relevant OWL-S files>
  tmp
  <any other OWL resource files>
```

  
#### 1.2 Setting Up a Local Server
OWLS2PDDL expects its services to be hosted on a webserver, and was developed with a python http server hosting these services. The CLI commands that follow will set up python 3’s http server.

Windows:
```
cd path/to/root/folder
python3 -m http.server 8080 --bind 127.0.0.1
```

Linux / MacOS:
```
cd path/to/root/folder
python3 -m http.server 8080 --bind 127.0.0.1
```

#### 1.3 Invoking OWLS2PDDL
  

Firstly, make sure to include the jsoup-1.13.1.jar as Referenced Libraries and check if it is located in the correct build path. Then, assuming that FastDownwardCaller.jar is on one's buildpath, it can be invoked as follows:
```java
import java.io.File;
import java.io.IOException;
import org.semanticweb.owl.model.OWLException;
import de.dfki.owls2pddxml_2_0.OWLS2PDDL;
	
	public static final String PDDXML_DOMAIN_PRINT_LOCATION = "your-path/domain.xml";
	public static final String PDDXML_PROBLEM_PRINT_LOCATION = "your-path/problem.xml";
	public static final String PDDL_DOMAIN_PRINT_LOCATION = "your-path/domain.pddl";
	public static final String PDDL_PROBLEM_PRINT_LOCATION = "your-path/problem.pddl";

		
	OWLS2PDDL converter = new OWLS2PDDL("exampleDomainName", "your-path-to-root-folder/Scallops_Input");
	
	File initFile = new File("path-to-inital-ontology/Simple_InitialOntology.owl");
	File goalFile = new File("path-to-Goal-ontology/Simple_GoalOntology.owl");
		

	try {
	
	converter.addToInitialState(initFile.toURI());
	converter.addToGoalState(goalFile.toURI());
			
	File[] files = new File("/Users/guangyichen/Desktop/Scallops_Input/services").listFiles();
			
	for (File file : files)
	if (!file.isDirectory() && file.getName().contains(".owl")) {
				
		converter.addServices(file.toURI()); 
				
	}
			
	} catch(OWLException | IOException e) {
	
	e.printStackTrace();
			
	}
		
	
	String pddlDomain = converter.getDomain().makePDDLTextDocument(false, converter.getFilesWithPddl());
	String pddlProblem = converter.getProblem().makePDDLTextDocument(false);

	System.out.println("printing");
		      
	FileWriter pddlDomainFileWriter;

	try {
		pddlDomainFileWriter = new FileWriter(PDDL_DOMAIN_PRINT_LOCATION);
		String pddlString = converter.getDomain().makePDDLTextDocument(false, converter.getFilesWithPddl());
		pddlDomainFileWriter.write(pddlString);
			pddlDomainFileWriter.close();
		} catch (IOException e) {
		
		e.printStackTrace();
	}

	FileWriter pddlProblemFileWriter;
	
	try {
	
	pddlProblemFileWriter = new FileWriter(PDDL_PROBLEM_PRINT_LOCATION);
	pddlProblemFileWriter.write(converter.getProblem().makePDDLTextDocument(false));
	pddlProblemFileWriter.close();
	} catch (IOException e) {
		    
		e.printStackTrace();
	}

	FileWriter domainFileWriter;

	try {
		domainFileWriter = new FileWriter(PDDXML_DOMAIN_PRINT_LOCATION);
		domainFileWriter.write(converter.getDomain().makeTextDocument());
		domainFileWriter.close();
	} catch (IOException e) {
	
	    e.printStackTrace();
	}
		      
	FileWriter problemFileWriter;
		      
	try {
		        
		problemFileWriter = new FileWriter(PDDXML_PROBLEM_PRINT_LOCATION);
		problemFileWriter.write(converter.getProblem().makeTextDocument());
	    problemFileWriter.close();
		   
	} catch (IOException e) {
	
		e.printStackTrace();
		
	}
	}
	
}
```
  

### 2. FastdownwardCaller
#### 2.1 Installing Fast-Downward
```
cd path/to/working-dir
git clone https://github.com/aibasel/downward
./build.py release
```


#### 2.2 Setting Up a Local Server
Windows:
```
cd path/to/root/folder
python3 -m http.server 8080 --bind 127.0.0.1
```

Linux / MacOS:
```
cd path/to/root/folder
python3 -m http.server 8080 --bind 127.0.0.1
```
  
  

#### 2.3 Invoking FastDownwardCaller V1.2
```java
import java.io.File;
import java.net.URI;

import org.jsoup.*;
import org.w3c.dom.Document;

import de.dfki.fastdownwardcaller.*;

	// initialize pointing to FastDownward directory
	FastDownwardCaller caller = new FastDownwardCaller("your-path/downward-main");
		
	try {
		
		// pass Strings representing domain and problem pddls 
		// here you may use your outputs from OWLS2PDDL in the previous step for problem.pddl and domain.pddl files
		caller.readInProblem(new File("your-path-to-problem_pddl/problem.pddl"));
		caller.readInDomain(new File("your-path-to-domain_pddl/domain.pddl"));
		
	} catch (Exception e) {
			
		e.printStackTrace();
			
	}
		
	// Get a composed service as a w3c Document
	Plan lowestCostPlan = caller.getLowestCostPlan(true);
		
	File outputDirectory = new File("/Users/guangyichen/Desktop/");
	URI compositeServiceUri =  outputDirectory.toURI();
		
	CompositeService actionToService = new CompositeService(lowestCostPlan, compositeServiceUri);
	Document composedService = actionToService.getDocument();
		
	}
}
```
  
  

## Complete Example
You may find a complete example included in this repository.
