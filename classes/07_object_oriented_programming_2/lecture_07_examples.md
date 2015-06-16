# Object-Oriented Programming, Examples

What follows are just some simple examples of OOP in Python.

## Reading Text Files

This is a simple utility class to make reading a text file a tiny bit easier.

    class TextFile(object):
    
        def __init__(self, filepath, ditch_header=False):
            self.filepath = filepath
            self.ditch_header = ditch_header
            self.lines = []
            self.read_file()
        
        def read_file(self):
            '''read the text file, with or without the first line
            and strip the right end of the file, if necessary'''
            # read the file
            f = open(self.filepath, 'r')
            lines = f.readlines()
            f.close()
    
            # do we want the header row?
            if self.ditch_header:
                start = 1
            else:
                start = 0
            
            # save the files lines as a class attribute
            self.lines = [line.rstrip() for line in lines[start:]]
        
        def number_of_lines(self):
            '''How many lines are in our file?'''
            return len(self.lines)

Notice that the `__init__` method takes two parameters: `filepath` is the full path of the file we want to read, and `ditch_header` is a boolean to say whether we want to remove the first line of the file or not. Also notice that the `__init__` method calls a method to fill one of it's values.

We have two methods in this class: `read_file` reads each line of the text file and saves the results (with or without the header), and `number_of_lines` which returns the number of lines in the file.

Now, let's look at another class. This time we will be reading a CSV file.

    class CSVFile(object):
    
        def __init__(self, filepath):
            self.filepath = filepath
            self.lines = []
            self.header = []
            self.read_file()
        
        def read_file(self):
            '''read the csv file, with or without the first line
            and strip the right end of the file, if necessary'''
            # read the file
            f = open(self.filepath, 'r')
            lines = f.readlines()
            f.close()
            
            # save off the CSV header
            self.header = lines[0].rstrip().split(',')
            
            # save the files lines as a class attribute
            self.lines = [line.rstrip().split(',') for line in lines[1:]]
        
        def number_of_lines(self):
            '''How many lines are in our file?'''
            return len(self.lines)
        
        def number_of_columns(self):
            '''How many columns are in our CSV file?'''
            return len(self.lines[0])

As usual, we see that reading a CSV file is a lot like reading a text file. Our `__init__` method is very similar, except it doesn't take a `ditch_header` parameter, and includes a `header` value. The `read_file` method is really similar, but we use `split(',')` to create lists instead of strings for each line. Also, we added a `number_of_columns` method, which returns the number of columns in our CSV file.

Both `TextFile` and `CSVFile` above are fine as is. But they share a lot of code and ideas (because CSV files are just a type of text file). And it would be nice if we didn't have to repeat ourselves, so maybe both of these classes should subclass the same abstract class:

    from abc import ABCMeta, abstractmethod
    
    class AbstractTextFile(object):
        __metaclass__ = ABCMeta
        
        def __init__(self, filepath, ditch_header):
            self.filepath = filepath
            self.ditch_header = ditch_header
            self.lines = []
            self.read_file()

        @abstractmethod
        def read_file(self):
            pass
            
        def number_of_lines(self):
            '''How many lines are in our file?'''
            return len(self.lines)

Now we can re-write `TextFile` using our new abstract class:

    class TextFile(AbstractTextFile):
    
        def __init__(self, filepath, ditch_header=False):
            AbstractTextFile.__init__(filepath, ditch_header)
        
        def read_file(self):
            '''read the text file, with or without the first line
            and strip the right end of the file, if necessary'''
            # read the file
            f = open(self.filepath, 'r')
            lines = f.readlines()
            f.close()
    
            # do we want the header row?
            if self.ditch_header:
                start = 1
            else:
                start = 0
            
            # save the files lines as a class attribute
            self.lines = [line.rstrip() for line in lines[start:]]

We can also re-write our `CSVFile` class using the same abstract base class:

    class CSVFile(AbstractTextFile):
    
        def __init__(self, filepath):
            AbstractTextFile.__init__(filepath, True)
            self.header = []
        
        def read_file(self):
            '''read the csv file, with or without the first line
            and strip the right end of the file, if necessary'''
            # read the file
            f = open(self.filepath, 'r')
            lines = f.readlines()
            f.close()
            
            # save off the CSV header
            self.header = lines[0].rstrip().split(',')
            
            # save the files lines as a class attribute
            self.lines = [line.rstrip().split(',') for line in lines[1:]]
        
        def number_of_columns(self):
            '''How many columns are in our CSV file?'''
            return len(self.lines[0])
        
Now we have two classes `TextFile` and `CSVFile` based on the same abstract class `AbstractTextFile`. There is some shared code and functionality in the abstract class (`AbstractTextFile`). But since CSV files are special text files, we have some differences there that we need to express. The goal of using this abstract class is to express the similarity between our two subclasses.

Abstract classes are particularly useful if you don't just have two classes in your code, but many. You can organize your thoughts around all of the things you need to do, and when you need to remember something, you can just look at the superclasses and subclasses of the class you're working on.


[Back to Lecture](lecture_07.md)