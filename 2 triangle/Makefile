opengllib = -lopengl32 -lglu32 -lgdi32

headers = ../headers/
libs = ../libs/
src = ../src/
glad_dir = ../src/glad.c

triangle : glad.o triangle.cpp
	g++ -static -o triangle triangle.cpp glad.o -I$(headers) -L$(libs) -lglfw3 $(opengllib)

glad.o : $(glad_dir)
	gcc -I$(headers) -c $(glad_dir)

.PHONY : clean

clean :
	rm glad.o Shader.o