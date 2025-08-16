const express = require('express');
const { createClient } = require('@supabase/supabase-js');
const app = express();
app.use(express.json());

// 🔑 Configuration Supabase (à remplacer)
const supabaseUrl = 'https://VOUS-ID.supabase.co';
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.xxxxx';
const supabase = createClient(supabaseUrl, supabaseKey);

// 🌍 GET /scholarships - avec filtres
app.get('/scholarships', async (req, res) => {
  try {
    let query = supabase.from('scholarships').select('*');

    if (req.query.country) query = query.eq('country', req.query.country);
    if (req.query.level) query = query.eq('level', req.query.level);
    if (req.query.q) {
      query = query.ilike('title', `%${req.query.q}%`);
    }

    const { data, error } = await query;
    if (error) throw error;

    res.json(data);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// 📥 POST /apply - soumettre une candidature
app.post('/apply', async (req, res) => {
  const { student_id, scholarship_id } = req.body;

  const { data, error } = await supabase
    .from('applications')
    .insert({
      student_id,
      scholarship_id,
      status: 'en cours',
      created_at: new Date().toISOString()
    });

  if (error) {
    return res.status(500).json({ error: error.message });
  }

  res.status(201).json({ message: 'Candidature enregistrée', data });
});

// 🏠 Page d’accueil
app.get('/', (req, res) => {
  res.send(`
    <h1>🚀 ScholarPath API v2</h1>
    <p>Connecté à Supabase ✅</p>
    <p>Test : <a href="/scholarships">/scholarships</a></p>
  `);
});

const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`API démarrée sur le port ${port}`);
}); 
