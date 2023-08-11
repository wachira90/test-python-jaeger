# test-python-jaeger
test-python-jaeger

# Running Jaeger in a Docker Container

##  First, install Jaeger Client on your machine:

```
pip install jaeger-client
```

## Now, let’s run Jaeger backend as an all-in-one Docker image. The image launches the Jaeger UI, collector, query, and agent:

```
docker run -d -p6831:6831/udp -p16686:16686 jaegertracing/all-in-one:latest
```

TIP:  To check if the docker container is running, use: Docker ps.

Once the container starts, open http://localhost:16686/  to access the Jaeger UI. The container runs the Jaeger backend with an in-memory store, which is initially empty, so there is not much we can do with the UI right now since the store has no traces. 

# Creating Traces on Jaeger UI

## Create a Python program to create Traces:

Let’s generate some traces using a simple python program. You can clone the Jaeger-Opentracing repository given below for a sample program that is used in this blog.

booking-mgr.py

```py
import sys
import time
import logging
import random
from jaeger_client import Config
from opentracing_instrumentation.request_context import get_current_span, span_in_context

def init_tracer(service):
    logging.getLogger('').handlers = []
    logging.basicConfig(format='%(message)s', level=logging.DEBUG)    
    config = Config(
        config={
            'sampler': {
                'type': 'const',
                'param': 1,
            },
            'logging': True,
        },
        service_name=service,
    )
    return config.initialize_tracer()

def booking_mgr(movie):
    with tracer.start_span('booking') as span:
        span.set_tag('Movie', movie)
        with span_in_context(span):
            cinema_details = check_cinema(movie)
            showtime_details = check_showtime(cinema_details)
            book_show(showtime_details)

def check_cinema(movie):
    with tracer.start_span('CheckCinema', child_of=get_current_span()) as span:
        with span_in_context(span):
            num = random.randint(1,30)
            time.sleep(num)
            cinema_details = "Cinema Details"
            flags = ['false', 'true', 'false']
            random_flag = random.choice(flags)
            span.set_tag('error', random_flag)
            span.log_kv({'event': 'CheckCinema' , 'value': cinema_details })
            return cinema_details

def check_showtime( cinema_details ):
    with tracer.start_span('CheckShowtime', child_of=get_current_span()) as span:
        with span_in_context(span):
            num = random.randint(1,30)
            time.sleep(num)
            showtime_details = "Showtime Details"
            flags = ['false', 'true', 'false']
            random_flag = random.choice(flags)
            span.set_tag('error', random_flag)
            span.log_kv({'event': 'CheckCinema' , 'value': showtime_details })
            return showtime_details

def book_show(showtime_details):
    with tracer.start_span('BookShow',  child_of=get_current_span()) as span:
        with span_in_context(span):
            num = random.randint(1,30)
            time.sleep(num)
            Ticket_details = "Ticket Details"
            flags = ['false', 'true', 'false']
            random_flag = random.choice(flags)
            span.set_tag('error', random_flag)
            span.log_kv({'event': 'CheckCinema' , 'value': showtime_details })
            print(Ticket_details)

assert len(sys.argv) == 2
tracer = init_tracer('booking')
movie = sys.argv[1]
booking_mgr(movie)
# yield to IOLoop to flush the spans
time.sleep(2)
tracer.close()
```

The Python program takes a movie name as an argument and calls three functions that get the cinema details, movie showtime details, and finally book a movie ticket.

It creates some random delays in all the functions to make it more interesting, as in reality the functions would take certain time to get the details. Also the function throws random errors to give us a feel of how the traces of a real-life application may look like in case of failures.

Here is a brief description of how OpenTracing has been used in the program:

### Initializing a tracer:

Init-Tracer.py

```
def init_tracer(service):
   logging.getLogger('').handlers = []
   logging.basicConfig(format='%(message)s', level=logging.DEBUG)   
   config = Config(
       config={
           'sampler': {
               'type': 'const',
               'param': 1,
           },
           'logging': True,
       },
       service_name=service,
   )
   return config.initialize_tracer()
```

### Using the tracer instance: 

tracer_instance.js

```
tracer = init_tracer('booking')
```

### Starting new child spans using start_span: 

tracer_start.js

```
with tracer.start_span('CheckCinema', child_of=get_current_span()) as span:
```

### Using Tags:

tags.js

```
span.set_tag('Movie', movie)
```

### Using Logs:

logs.js

```
span.log_kv({'event': 'CheckCinema' , 'value': cinema_details })
```

## Run the python program:‍

run_python.sh

```
$ python booking-mgr.py <movie-name>

Initializing Jaeger Tracer with UDP reporter
Using sampler ConstSampler(True)
opentracing.tracer initialized to <jaeger_client.tracer.Tracer object at 0x7f72ffa25b50>[app_name=booking]
Reporting span cfe1cc4b355aacd9:8d6da6e9161f32ac:cfe1cc4b355aacd9:1 booking.CheckCinema
Reporting span cfe1cc4b355aacd9:88d294b85345ac7b:cfe1cc4b355aacd9:1 booking.CheckShowtime
Ticket Details
Reporting span cfe1cc4b355aacd9:98cbfafca3aa0fe2:cfe1cc4b355aacd9:1 booking.BookShow
Reporting span cfe1cc4b355aacd9:cfe1cc4b355aacd9:0:1 booking.booking
```


https://www.jaegertracing.io/docs/1.9/

https://opentracing.io/docs/



