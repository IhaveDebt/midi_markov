/**
 * MIDI-like Markov Note Generator (midi_markov.ts)
 *
 * Builds a Markov chain over note pitches and generates sequences.
 * This is a prototype that outputs ASCII note sequences (no MIDI file lib).
 *
 * Run: ts-node src/midi_markov.ts
 */

type Note = number; // MIDI pitch 0-127

class MarkovMusic {
  order: number;
  chains: Map<string, Map<number, number>> = new Map();

  constructor(order = 1) { this.order = order; }

  private key(seq: Note[]) { return seq.join(','); }

  train(seqs: Note[][]) {
    for (const seq of seqs) {
      for (let i = 0; i + this.order < seq.length; i++) {
        const state = this.key(seq.slice(i, i + this.order));
        const next = seq[i + this.order];
        const map = this.chains.get(state) || new Map();
        map.set(next, (map.get(next) || 0) + 1);
        this.chains.set(state, map);
      }
    }
  }

  private sampleNext(map: Map<number, number>) {
    const total = Array.from(map.values()).reduce((a, b) => a + b, 0);
    let r = Math.random() * total;
    for (const [k, v] of map) {
      if (r <= v) return k;
      r -= v;
    }
    return Array.from(map.keys())[0];
  }

  generate(seed: Note[], length = 32) {
    const result: Note[] = seed.slice();
    while (result.length < length) {
      const state = this.key(result.slice(result.length - this.order, result.length));
      const map = this.chains.get(state);
      if (!map) break;
      result.push(this.sampleNext(map));
    }
    return result;
  }
}

// demo with simple scales
if (require.main === module) {
  const mm = new MarkovMusic(2);
  const scale = [60, 62, 64, 65, 67, 69, 71, 72];
  const seqs = Array.from({ length: 50 }, () => {
    // random walks on scale
    const s = [];
    let idx = Math.floor(Math.random() * scale.length);
    for (let i = 0; i < 50; i++) {
      idx = Math.max(0, Math.min(scale.length - 1, idx + (Math.random() > 0.5 ? 1 : -1)));
      s.push(scale[idx]);
    }
    return s;
  });
  mm.train(seqs);
  const seed = [60, 62];
  console.log('Generated notes:', mm.generate(seed, 32).join(' '));
}
