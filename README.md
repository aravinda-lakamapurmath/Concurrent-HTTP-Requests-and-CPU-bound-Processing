# Concurrent HTTP Requests and CPU-bound Processing

This project demonstrates how to achieve concurrent HTTP requests to 100,000 external API endpoints and process the data concurrently with some CPU-bound computations. The target SLA for the entire process of one API request and processing needs to be within 500ms. The external API is limited to 20 requests per second.

## Requirements

- Python 3.8+
- `aiohttp` for asynchronous HTTP requests
- `asyncio` for asynchronous programming
- `concurrent.futures` for parallel CPU-bound processing

## Installation

1. Clone the repository:

    ```bash
    git clone https://github.com/yourusername/concurrent-requests.git
    cd concurrent-requests
    ```

2. Install the required libraries:

    ```bash
    pip install aiohttp
    ```

## Usage

### Configuration

- `API_URL`: The URL of the external API endpoint.
- `API_LIMIT`: The maximum number of requests per second (rate limit).
- `TOTAL_REQUESTS`: The total number of requests to be made (100,000).

### Code

```python
import aiohttp
import asyncio
import time
from concurrent.futures import ProcessPoolExecutor
from typing import List

API_URL = "https://example.com/api"
API_LIMIT = 20
TOTAL_REQUESTS = 100000

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.json()

def cpu_bound_task(data):
    # Simulate CPU-bound processing
    result = sum(x * x for x in range(10000))
    return result

async def bounded_fetch(sem, session, url):
    async with sem:
        data = await fetch(session, url)
        return data

async def main():
    sem = asyncio.Semaphore(API_LIMIT)
    async with aiohttp.ClientSession() as session:
        tasks = []
        for _ in range(TOTAL_REQUESTS):
            task = asyncio.ensure_future(bounded_fetch(sem, session, API_URL))
            tasks.append(task)
        responses = await asyncio.gather(*tasks)
    
    with ProcessPoolExecutor() as executor:
        loop = asyncio.get_event_loop()
        futures = [loop.run_in_executor(executor, cpu_bound_task, data) for data in responses]
        await asyncio.gather(*futures)

if __name__ == "__main__":
    start_time = time.time()
    asyncio.run(main())
    end_time = time.time()
    print(f"Total time taken: {end_time - start_time:.2f} seconds")
# PROJECT
