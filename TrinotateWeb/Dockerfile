FROM debian:buster-slim

ENV TRINOTATE_HOME=/app/Trinotate
ENV TRINOTATE_VERSION=3.1.1

WORKDIR /app

RUN apt-get -y update && \
    apt-get install -y lighttpd libhtml-template-perl libdbd-sqlite3-perl && \
    rm -rf /var/lib/apt/lists/*

# Not required, using Debian packages instead
# RUN apt-get install -y cpanminus build-essential
# RUN cpanm -i DBI && \
#     cpanm -i HTML && \
#     cpanm -i HTML::Template && \
#     cpanm -i DBD::SQLite

ADD https://github.com/Trinotate/Trinotate/archive/Trinotate-v${TRINOTATE_VERSION}.tar.gz Trinotate-v${TRINOTATE_VERSION}.tar.gz
RUN tar xvzf Trinotate-v${TRINOTATE_VERSION}.tar.gz && \
    rm Trinotate-v${TRINOTATE_VERSION}.tar.gz && \
    mv Trinotate-Trinotate-v${TRINOTATE_VERSION} Trinotate

COPY TrinotateAnno.sqlite /data/TrinotateAnno.sqlite 
COPY lighttpd.conf /app/lighttpd.conf

RUN chown -R www-data:www-data /app

EXPOSE 80

ENTRYPOINT ["lighttpd", "-D", "-f", "/app/lighttpd.conf"]
