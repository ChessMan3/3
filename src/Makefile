mkfile_path := $(abspath $(lastword $(MAKEFILE_LIST)))
current_dir := $(notdir $(patsubst %/,%,$(dir $(mkfile_path))))
CFISH= $(current_dir:.d=)
CC=clang
LDFLAGS=
STRIP=strip

default:
	$(MAKE) -j quick
help:
	@echo "type make"
	@echo ""

clang:
	@/opt/local/libexec/llvm-3.9/bin/llvm-profdata merge -output=engine.profdata *.profraw
	$(MAKE) -j target=UNIX \
	CC=clang \
	opt='-DUNIX -DUSE_POPCNT -DNDEBUG -DIS_64BIT -DAPPLE' \
	CFLAGS='-Wall -m64 -arch x86_64 -mmacosx-version-min=10.9 -pthread -std=c11 -mtune=native \
	-march=native -pedantic -Wextra -Wshadow -O3 -mdynamic-no-pic \
     -mssse3 -mpopcnt -fprofile-instr-use=engine.profdata' \
	LDFLAGS='$(LDFLAGS) -fprofile-instr-use=engine.profdata' \
	cfish-make

quick:
	$(MAKE) -j target=UNIX \
	CC=clang \
	opt='-DUNIX -DUSE_POPCNT -DNDEBUG -DIS_64BIT -DAPPLE' \
	CFLAGS='-Wall -m64 -arch x86_64 -mmacosx-version-min=10.9 -pthread -std=c11 -mtune=native \
	-march=native -pedantic -Wextra -Wshadow -O3 -mdynamic-no-pic \
	-mssse3 -mpopcnt ' \
	LDFLAGS='$(LDFLAGS) ' \
	cfish-make

clang-profile:
	$(MAKE) -j target=x86_64 \
	CC=clang \
	opt='-DUNIX -DUSE_POPCNT -DNDEBUG -DIS_64BIT -DAPPLE' \
	CFLAGS='-Wall -m64 -arch x86_64 -mmacosx-version-min=10.9 -pthread -std=c11 -mtune=native \
    -march=native -pedantic -Wextra -Wshadow -Os -O3 -mdynamic-no-pic \
	-msse4.2 -mpopcnt -fprofile-instr-generate ' \
	LDFLAGS='$(LDFLAGS)  -fprofile-instr-generate  ' \
	cfish-make

cpro:
	@rm -rf *.o
	@rm -rf log.*
	@rm -rf game.*
	@rm -rf prof
	@rm -rf *.profraw
	@rm -rf *.profdata
	@rm -rf $(CFISH)
	@mkdir prof
	@touch *.c  *.h
	$(MAKE) -j clang-profile
	@echo "#!/bin/csh" > runprof
	@echo "./$(CFISH) <<EOF" >>runprof
	@echo "bench 16 1 10" >>runprof
	@echo "quit" >>runprof
	@echo "EOF" >>runprof
	@chmod +x runprof
	@./runprof
	@rm runprof
	@touch *.c *.h
	$(MAKE) -j clang
	$(STRIP) $(CFISH)

objects =  search.o bitboard.o evaluate.o movegen.o  bitbase.o movepick.o benchmark.o \
endgame.o  main.o material.o misc.o  pawns.o position.o psqt.o tbprobe.o thread.o \
timeman.o tt.o uci.o ucioption.o numa.o settings.o

# Do not change anything below this line

opts = $(opt) -D$(target)

#	@$(MAKE) -j opt='$(opt)'  CFLAGS='$(CFLAGS)' $(CFISH)
cfish-make:
	@$(MAKE) opt='$(opt)'  CFLAGS='$(CFLAGS)' $(CFISH)

cfish.o: *.c *.h

$(CFISH):	$(objects)
	$(CC) $(LDFLAGS) -o $(CFISH) $(objects) -lm  $(LIBS)

clean:
	-rm -f *.o $(CFISH)

$(objects):

.c.o:
	$(CC) $(CFLAGS) $(opts) -c $*.c

