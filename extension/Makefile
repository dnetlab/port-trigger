
LIB 	:= libipt_TRIGGER.so
SOURCES := libipt_TRIGGER.c
OBJS 	:= $(SOURCES:.c=.o)

CFLAGS  += -Os -pipe
LDFLAGS += -lm
IFLAGS  += -I./

all: $(LIB)

$(LIB): $(OBJS)
	$(CC) -s -shared -o $@ $(OBJS) $(LDFLAGS)

$(OBJS): $(SOURCES)
	$(CC) -c -o $@ -fpic $(CFLAGS) $(IFLAGS) $< 

clean:
	rm -f $(OBJS) $(LIB) 

.PHONY: clean all
