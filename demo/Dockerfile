FROM alpine:3.7
COPY . /demo
WORKDIR /demo
RUN apk add --no-cache \
        uwsgi-python3 \
        python3 && \
    pip3 install --no-cache-dir -r requirements.txt
EXPOSE 7000
CMD ["uwsgi", "-i", "uwsgi.ini"]
