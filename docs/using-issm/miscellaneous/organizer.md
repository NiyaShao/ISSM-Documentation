# organizer

ISSM organizer is a module that supports automatic saving and loading of multiple runs as (.md) files. It can save and load each step of a model and organize different model runs.

### initiating an organizer

The method starts by creating an instance of the organizer class. For example, the following code builds an organizer instance, where the model files will be stored in the ‘./Models/’ folder with prefix ‘AGU2015’. The prefix can be used to conveniently organize different model runs. The ‘steps’ parameter needs a list of numerical id as an input to tell the organizer which steps will be run.

```
org = organizer('repository',['Models/'],...
                'prefix',['AGU2015'],...
                'steps',[1,2]);  
%build an empty organizer object with a given repository
```

To check the current state of ‘org’, disp() can be used. For example, disp(org) immediately after creating the above organizer will results in

```
disp(org)                                        
   Repository: 'Models/'
   Prefix:     'AGU2015'
   Color:      '41;37'
   skipio:     0
   no step
```

### Organizing steps in a model run

After the organizer is created, it can associate step names (strings) to each id in the ‘steps’

To perform a step in the model using an organizer, use the function *perform*. The function *perform* will take in the organizer and a string as the name of the step. The function will advance the current step id by 1 and associate the step name with the current id. If the current step id doesn’t exist, then the current step id will be set to 1, and step id 1 will be associated with the step name. The function will return true if the step name has not been used before and the id associated with the step name is in the ‘steps’ input when creating the organizer

For example, the following code will map id ‘1’ to string ‘Mesh’. If the ‘steps’ used to create the organizer contain 1, then *perform(org, ‘Mesh’)* will return True, and the code inside the if block will be performed. If the *steps* does not contain 1, then this code will be skipped.

```
if perform(org,'Mesh'),
	md=triangle(model,['./Data/GlacierDomain.exp'],100); %create a model
        ...
	savemodel(org,md); %save the model
else
	disp(‘Mesh step skipped’)
end
```

Then if-clause can run the steps specified when creating the organizer. In this code, because the *steps* is [2], the first if block will not be runned, and the second if block will be runned.

```
org = organizer('repository',['Models/'],'prefix',['AGU2015'],'steps',[2]);  
% build an empty organizer object with a given repository

% Associating id 1 with ‘Mesh’. 
% Id 1 is not in ‘steps’, return False and skip the if block
if perform(org,'Mesh'),
	md=triangle(model,['./Data/GlacierDomain.exp'],100); %create a model
        ...
	savemodel(org,md); %save the model
else
	disp(‘Mesh step skipped’)
end

%associating id 2 with ‘Parameterization’. Id 2 is in the ‘steps’, return true and execute the if block
if perform(org,'Parameterization'), 
	md=loadmodel(org,’Mesh’)
        ...
	savemodel(org,md); %save the model
else
	disp(‘Mesh step skipped’)
end
```

Using this structure, the steps to create a model are modularized, and you can start from any steps while keeping the previous steps saved.

You can check the id and name strings of the step that have been runned with the organizer using:

```
org.steps.id
org.steps.string
```

Can check the current step the organizer is on using:

```
org.currentstep
```
### Saving and loading models

The function *savemodel* takes in the organizer and model, to save the model with the name prefix defined when creating the organizer and the step name. For example, in the code above, the *savemodel(org,md);* will save the model in the filepath ‘./Models/AGU2015_Mesh.mat’

The function *loadmodel* takes in the organizer and the step name of previously saved moded. For example, in the code above, *md=loadmodel(org,’Mesh’)* will load the model saved in the filepath ‘./Models/AGU2015_Mesh.mat’

### Running different experiments with the organizer

The following code creates different model names based on the control variables (topo_code, fric_type, inversion_on). An outer loop to run the script can easily iterate through each designed experiment (e.g. for i = fric_type: … end), and the output will be organized with different naming conventions.

```
for topo_code = 1:3
  for fric_type = 1:5
    for inversion_on = [False,True]
    
      org=organizer('repository',['./Models'],...
      'prefix',['Thwaites_topo' num2str(topo_code) '_fric' num2str(fric_type) '_inv' num2str(inversion_on) '_'],...
      'steps',steps); clear steps;
			
      if perform(org,’Mesh’)
        ...
      end

      if perform(org,’Parameterization’)
        if topo_code == 1
        ...
        end
      end

    end
  end
end
```



