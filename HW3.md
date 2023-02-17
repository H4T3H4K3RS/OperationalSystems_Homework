# Задание 1

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>

#define BUF_SIZE 4096
#define CIRC_BUF_SIZE 1024

typedef struct {
    char buffer[CIRC_BUF_SIZE];
    size_t head;
    size_t tail;
    size_t size;
} CircularBuffer;

void circ_buf_push(CircularBuffer *circ_buf, char c) {
    circ_buf->buffer[circ_buf->head] = c;
    circ_buf->head = (circ_buf->head + 1) % CIRC_BUF_SIZE;
    if (circ_buf->size < CIRC_BUF_SIZE) {
        circ_buf->size++;
    } else {
        circ_buf->tail = (circ_buf->tail + 1) % CIRC_BUF_SIZE;
    }
}

void circ_buf_flush(CircularBuffer *circ_buf, int fd) {
    if (circ_buf->size > 0) {
        if (circ_buf->tail <= circ_buf->head) {
            write(fd, circ_buf->buffer + circ_buf->tail, circ_buf->head - circ_buf->tail);
        } else {
            write(fd, circ_buf->buffer + circ_buf->tail, CIRC_BUF_SIZE - circ_buf->tail);
            write(fd, circ_buf->buffer, circ_buf->head);
        }
        circ_buf->size = 0;
        circ_buf->head = 0;
        circ_buf->tail = 0;
    }
}
int main(int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(stderr, "Usage: %s <sourcefile> <destfile>\n", argv[0]);
        exit(EXIT_FAILURE);
    }
    int source_fd, dest_fd;
    ssize_t num_read;
    char buf[BUF_SIZE];
    CircularBuffer circ_buf = {.head = 0, .tail = 0, .size = 0};
    struct stat st{};
    if (stat(argv[1], &st) == -1) {
        perror("stat");
        exit(EXIT_FAILURE);
    }
    if ((st.st_mode & S_IXUSR) || (st.st_mode & S_IXGRP) || (st.st_mode & S_IXOTH)) {
        source_fd = open(argv[1], O_RDONLY);
        if (source_fd == -1) {
            perror("open");
            exit(EXIT_FAILURE);
        }
        dest_fd = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC,
                       S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH);
        if (dest_fd == -1) {
            perror("open");
            exit(EXIT_FAILURE);
        }
        while ((num_read = read(source_fd, buf, BUF_SIZE)) > 0) {
            if (write(dest_fd, buf, num_read) != num_read) {
                perror("write");
                exit(EXIT_FAILURE);
            }
        }
        if (num_read == -1) {
            perror("read");
            exit(EXIT_FAILURE);
        }
        if (close(source_fd) == -1) {
            perror("close source");
            exit(EXIT_FAILURE);
        }
        if (close(dest_fd) == -1) {
            perror("close dest");
            exit(EXIT_FAILURE);
        }
        return 0;
    }
    source_fd = open(argv[1], O_RDONLY);
    if (source_fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }
    dest_fd = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC,
                   S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH);
    if (dest_fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }
    while ((num_read = read(source_fd, buf, BUF_SIZE)) > 0) {
        for (ssize_t i = 0; i < num_read; i++) {
            char c = buf[i];
            if (c >= ' ' && c <= '~') {
                circ_buf_push(&circ_buf, c);
            } else {
                circ_buf_flush(&circ_buf, dest_fd);
                if (write(dest_fd, &c, 1) != 1) {
                    perror("write");
                    exit(EXIT_FAILURE);
                }
            }
            if (circ_buf.size == CIRC_BUF_SIZE) {
                circ_buf_flush(&circ_buf, dest_fd);
            }
        }
    }
    if (num_read == -1) {
        perror("read");
        exit(EXIT_FAILURE);
    }
    circ_buf_flush(&circ_buf, dest_fd);
    if (close(source_fd) == -1) {
        perror("close source");
        exit(EXIT_FAILURE);
    }
    if (close(dest_fd) == -1) {
        perror("close dest");
        exit(EXIT_FAILURE);
    }
    return 0;
}
```
