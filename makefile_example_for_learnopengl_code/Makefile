headers = ../headers/
libs = ../libs/
src = ../src/
opengllib = -lopengl32 -lglu32 -lgdi32

assimp_path = ../libs/

skybox : cubemaps_skybox.cpp stb_image.o glad.o
	g++ -static -o skybox cubemaps_skybox.cpp stb_image.o glad.o -I$(headers) -L$(libs) -lglfw3 -lassimp $(opengllib)

stb_image.o : $(src)stb_image.cpp
	g++ -I$(headers) -o stb_image.o -c $(src)stb_image.cpp

glad.o : $(src)glad.c
	gcc -I$(headers) -c $(src)glad.c

.PHONY : clean

clean :
	rm glad.o