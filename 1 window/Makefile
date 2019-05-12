opengllib = -lopengl32 -lglu32 -lgdi32

headers = ../headers
libs = ../libs
glad_dir = ../src/glad.c

window : glad.o window.cpp
	g++ -o window window.cpp glad.o -I$(headers) -L$(libs) -lglfw3 $(opengllib)

glad.o : $(glad_dir)
	gcc -I$(headers) -c $(glad_dir)

.PHONY : clean

clean :
	rm glad.o