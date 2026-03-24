# RANDOMJOKEGENERATOR
const express = require('express');
const axios = require('axios');
const cors = require('cors');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.static('public'));

// API route to fetch a random joke
app.get('/api/joke', async (req, res) => {
  try {
    const jokeType = req.query.type || 'any'; // 'any', 'programming', 'general', 'knock-knock', etc.
    
    // Using JokeAPI - supports multiple categories
    const response = await axios.get(`https://v2.jokeapi.dev/joke/${jokeType}`);
    
    if (response.data.error) {
      return res.status(404).json({ error: 'No joke found' });
    }

    // Handle both single and two-part jokes
    const joke = {
      type: response.data.type,
      category: response.data.category,
      content: response.data.type === 'single' 
        ? response.data.joke 
        : `${response.data.setup}\n\n${response.data.delivery}`
    };

    res.json(joke);
  } catch (error) {
    console.error('Error fetching joke:', error.message);
    res.status(500).json({ error: 'Failed to fetch joke. Try again!' });
  }
});

// Route to get available joke categories
app.get('/api/categories', async (req, res) => {
  try {
    const response = await axios.get('https://v2.jokeapi.dev/categories');
    res.json(response.data);
  } catch (error) {
    console.error('Error fetching categories:', error.message);
    res.status(500).json({ error: 'Failed to fetch categories' });
  }
});

// Start server
app.listen(PORT, () => {
  console.log(`🎭 Joke Generator running on http://localhost:${PORT}`);
});
