# OWLS-FDplan

## Introduction
OWLS-FDplan is a service composition planner: Given a set of problem and/or domain described in OWL as input, as well as a service composition problem expressed in OWL-S 1.1, OWLS-FDplan produces a set of plans solving that problem. 

It consists of two programs (as submodules in this repository): 
1) using [OWLS2PDDL](https://github.com/garrison-chen/OWLS2PDDL/tree/fb1e1729b549b74075b8125f749d953b1a0efaf3) to convert OWL files into PDDL files; 
2) invoking [FastDownwardCaller](https://github.com/garrison-chen/FastDownwardCaller/tree/a39906beb93233eb077adcd96f0c53f3bbb727d4) on this PDDL representation to produce the set of plans (OWL-S).


## How to run
### 1. OWL2SPDDL
#### 1.1 Local File System
The current version of OWLS2PDDL requires read/write access to a local directory. This directory must be hosted at http://127.0.0.1:8080, and should have the following structure:

```
root folder
  services
    <all relevant OWL-S files>
  Initial.owl file
  Goal.owl file
  tmp
```

  
#### 1.2 Setting Up a Local Server
OWLS2PDDL expects its services to be hosted on a webserver, and was developed with a python http server hosting these services. The CLI commands that follow will set up python 3’s http server.

Windows:
```bash
cd path\to\root\folder
python3 -m http.server 8080 --bind 127.0.0.1
```

Linux / MacOS:
```bash
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
	}
	
}
```
  

### 2. FastdownwardCaller
#### 2.1 Installing Fast-Downward
```bash
cd path/to/working-dir
git clone https://github.com/aibasel/downward
./build.py release
```


#### 2.2 Setting Up a Local Server
Windows:
```bash
cd path\to\root\folder
python3 -m http.server 8080 --bind 127.0.0.1
```

Linux / MacOS:
```bash
cd path/to/root/folder
python3 -m http.server 8080 --bind 127.0.0.1
```
  
  

#### 2.3 Invoking FastDownwardCaller
```java
import java.io.File;
import java.net.URI;

import org.jsoup.*;
import org.w3c.dom.Document;

import de.dfki.fastdownwardcaller.*;

	// initialize pointing to FastDownward directory
	FastDownwardCaller caller = new FastDownwardCaller("your-path/downward-main");
		
	try {
		caller.readInProblem(new File("your-path-to-problem_pddl/problem.pddl"));
		caller.readInDomain(new File("your-path-to-domain_pddl/domain.pddl"));
		
	} catch (Exception e) {
			
		e.printStackTrace();
			
	}
		
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



## Acknowledgement
This software is originally adapted and developed by [Anthony Heggen](https://www.linkedin.com/in/anthony-heggen-592188142/), and currently maintained by [Guangyi Chen](https://www.dfki.de/en/web/about-us/employee/person/guch01/).

Contact: Dr. Elena Jaramillo, [PD Dr. Matthias Klusch](https://www.dfki.de/~klusch/)
