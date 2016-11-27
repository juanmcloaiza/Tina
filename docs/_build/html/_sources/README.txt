.. _readme:

Using Tina
=================


Launching a Tina job is pretty simple, provided that a hadoop cluster is already properly 
mounted and files stored in HDFS using only **one block per file**:

::

	$ $HADOOP_HOME/bin/hadoop jar Tina.jar \
	  -${generic_hadoop_mapreduce_options} \
	  -files file1,file2,file3... \
	  -input input1,input2,input3... \
	  -output hdfs_output_dir \
	  -mapper your_mapper_script \
	  -reducer your_reducer_script
	
::

Compare to hadoop-streaming:

::

	$ $HADOOP_HOME/bin/hadoop jar hadoop-streaming.jar \
	  -${generic_hadoop_mapreduce_options} \
	  -files your_mapper_scrit,your_reducer_script,file1,file2,file3... \
	  -input input1,input2,input3... \
	  -output hdfs_output_dir \
	  -mapper your_mapper_script \
	  -reducer your_reducer_script
	  
::


For **Tina** to do its job, both ``your_mapper_script`` and ``your_reducer_script`` must be executables 
that take as arguments two and one  strings respectively, i.e., running them locally from the command line should look like:

::

	$ your_mapper_script m_arg1 m_arg2
	$ map_result_filename
	
::

and

::

	$ your_reducer_script r_arg1
	$ reduce_result_filename
	
::

In the logic of Tina, 

* ``m_arg1`` will represent an hdfs fiename (extracted from the paths after the ``-input`` flag, they are treated just as a dummy name by the script. You may want to use these dummy names to give a name to your mapper output).

* ``m_arg2`` will be an actual local file path on which ``your_mapper_script`` will operate (the stored block in the local filesystem where the mapper is running (known bug here: block filenames don't have extensions!). 

* As a result of themapping process, ``your_mapper_script`` should 1) write a file of any kind to the current working directory,  and 2) write a string to stdout with its filename (``map_result_filename`` above).


* ``your_reducer_script`` should instead be ready to read lines of text from a file ``r_arg1``. These lines will be no other thing than each of the ``map_result_filename`` emmited by the mappers.

* Once ``your_reducer_script`` knows all of these filenames, it will be able to combine them and produce 1) a single file of any kind saved to the current working directory and 2) a string ``reduce_result_filename`` written to stdout. 

After Tina has finished running, if the job was successful, you should be able to see
the results (both map and reduce results are kept) in the directory you designed as output, ``hdfs_output_dir``.
