FROM python:3.6-alpine
WORKDIR /opt
RUN pip install flask==1.1.2
EXPOSE 8080
ARG odoo
ARG pgadmin
ENV ODOO_URL $odoo
ENV PGADMIN_URL $pgadmin
COPY app.py ic-webapp/
COPY templates/ ic-webapp/templates/
COPY static/ ic-webapp/static/
COPY images/ ic-webapp/images/
ENTRYPOINT ["python", "ic-webapp/app.py"]