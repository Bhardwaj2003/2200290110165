# 2200290110165
const express = require('express');
const axios = require('axios');

const app = express();
const PORT = 9876;
const MAX_WINDOW_SIZE = 10;
const TIMEOUT_MS = 500;
const API_HOST = 'http://20.244.56.144/evaluation-service';

let slidingWindow = [];

const typeToPath = {
  p: 'primes',
  f: 'fibo',
  e: 'even',
  r: 'rand'
};

app.get('/numbers/:type', async (req, res) => {
  const { type } = req.params;
  const apiPath = typeToPath[type];

  if (!apiPath) {
    return res.status(400).json({ error: 'Unknown number type specified' });
  }

  const previousWindow = [...slidingWindow];
  let receivedNumbers = [];

  try {
    const apiResponse = await axios.get(`${API_HOST}/${apiPath}`, {
      timeout: TIMEOUT_MS,
      headers: {
        Authorization: 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJNYXBDbGFpbXMiOnsiZXhwIjoxNzQ2ODAwMjQ2LCJpYXQiOjE3NDY3OTk5NDYsImlzcyI6IkFmZm9yZG1lZCIsImp0aSI6IjZjZTEwNGI0LWEzOTMtNDZjZi1hY2U2LTY5YmNkZDdlMjkxMyIsInN1YiI6ImFrc2hhdC4yMjI2ZWMxMTg1QGtpZXQuZWR1In0sImVtYWlsIjoiYWtzaGF0LjIyMjZlYzExODVAa2lldC5lZHUiLCJuYW1lIjoiYWtzaGF0IHRvbWFyIiwicm9sbE5vIjoiMjIwMDI5MDEyMDAyMSIsImFjY2Vzc0NvZGUiOiJTeFZlamEiLCJjbGllbnRJRCI6IjZjZTEwNGI0LWEzOTMtNDZjZi1hY2U2LTY5YmNkZDdlMjkxMyIsImNsaWVudFNlY3JldCI6InlRYkpNRGNnclhZeFB4VFcifQ.2QBDPg3r61HjkgvqtRKqZvMPN6Vm84ADa-jD0W8t6Q0'
      }
    });

    if (apiResponse.data && Array.isArray(apiResponse.data.numbers)) {
      receivedNumbers = apiResponse.data.numbers;
      const newUniqueNumbers = receivedNumbers.filter(n => !slidingWindow.includes(n));
      slidingWindow.push(...newUniqueNumbers);

      if (slidingWindow.length > MAX_WINDOW_SIZE) {
        slidingWindow = slidingWindow.slice(-MAX_WINDOW_SIZE);
      }
    }

  } catch (err) {
    console.error(`Failed to fetch from /${apiPath}:`, err.message);
  }

  const average = slidingWindow.length
    ? parseFloat((slidingWindow.reduce((sum, val) => sum + val, 0) / slidingWindow.length).toFixed(2))
    : 0;

  res.json({
    previousWindow,
    currentWindow: slidingWindow,
    fetched: receivedNumbers,
    average
  });
});

app.listen(PORT, () => {
  console.log(`Number averaging service is live at http://localhost:${PORT}`);
});
