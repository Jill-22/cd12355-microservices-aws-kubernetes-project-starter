FROM python:3.12-slim

WORKDIR /app

# Install system packages required to build Python dependencies and connect to PostgreSQL
RUN apt-get update -y && apt-get install -y build-essential libpq-dev && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

# Upgrade packaging tools and install Python dependencies
RUN pip install --upgrade pip setuptools wheel && pip install -r requirements.txt

COPY . .

EXPOSE 5153

CMD ["python", "app.py"]
