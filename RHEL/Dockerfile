# vim:set ft=dockerfile:
# 
# Copyright The CloudNativePG Contributors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# 

ARG POSTGRES_VERSION=13
ARG BASE_IMG=rockylinux:8

FROM $BASE_IMG AS build
ARG POSTGRES_VERSION

RUN set -xe; \
	dnf module enable -y postgresql:${POSTGRES_VERSION}; \
	dnf install -y libpq-devel gcc python39 python39-devel

COPY requirements.txt /

RUN python3.9 -m venv /opt/barman-cloud; \
	/opt/barman-cloud/bin/pip3 install --upgrade pip setuptools; \
	/opt/barman-cloud/bin/pip3 install psycopg2; \
# Install barman-cloud
# TODO: Remove --no-deps once https://github.com/pypa/pip/issues/9644 is solved
	/opt/barman-cloud/bin/pip3 install --no-deps -r requirements.txt; \
	/opt/barman-cloud/bin/pip3 cache purge

FROM $BASE_IMG
ARG POSTGRES_VERSION
ARG PGAUDIT_QUALIFIER

RUN set -xe; \
	dnf module enable -y postgresql:${POSTGRES_VERSION}; \
#	dnf update -y; \
	dnf install --setopt install_weak_deps=False -y \
		postgresql-server \
		pgaudit \
		python39; \
	dnf clean all;

# retrieve the virtualenv from the build stage
COPY --from=build /opt/barman-cloud /opt/barman-cloud

ENV PATH="${PATH}:/opt/barman-cloud/bin"

# Change the uid of postgres to 26
RUN usermod -u 26 postgres
USER 26
