# THERMOACOUSTIC INSTABILITY — TOPOLOGICAL PHASE GEOMETRY
# Mathematical simulation of acoustic limit-cycle via phase transition

try:
    import google.colab
    IN_COLAB = True
    SAVE_PATH = '/content/'
except ImportError:
    IN_COLAB = False
    SAVE_PATH = ''

import numpy as np
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
import matplotlib.animation as animation
from scipy.integrate import odeint

# --- 1. Topological Engine Parameters ---
N = 30
np.random.seed(42)
sigma_omega = 0.25
omega_q = np.random.normal(0, sigma_omega, N)

# Critical Phase Threshold
K_c = 2 * np.std(omega_q) / np.pi

# Rocket Engine Scenarios (Reversed Logic from Qubits)
K_resonance = K_c * 4.00  # Destructive (r -> 1)
K_stable = K_c * 0.40     # Safe (r -> 0)

print("="*50)
print("TOPOLOGICAL PHASE METRIC ENGINE: ROCKET PROPULSION")
print("="*50)
print(f"System size: {N} oscillators (Pressure/Heat nodes)")
print(f"Critical Coupling Threshold (K_c): {K_c:.4f}")

def kuramoto(theta, t, omega, K, N):
    return omega + K/N * np.array([
        np.sum(np.sin(2*np.pi*(theta - theta[i]))) for i in range(N)])

theta0 = np.random.uniform(0, 1, N)

# --- 2. Simulation ---
NF = 120
t_sim = np.linspace(0, 40, NF)
sa_res = odeint(kuramoto, theta0.copy(), t_sim, args=(omega_q, K_resonance, N))
sa_stab = odeint(kuramoto, theta0.copy(), t_sim, args=(omega_q, K_stable, N))

# Calculate Order Parameter r(t)
r_res = np.array([np.abs(np.mean(np.exp(2j*np.pi*sa_res[i]))) for i in range(NF)])
r_stab = np.array([np.abs(np.mean(np.exp(2j*np.pi*sa_stab[i]))) for i in range(NF)])

print(f"\nRESULTS:")
print(f"Scenario A (K/K_c = 4.0): Final Coherence r = {r_res[-1]:.4f} -> DESTRUCTIVE RESONANCE")
print(f"Scenario B (K/K_c = 0.4): Final Coherence r = {r_stab[-1]:.4f} -> STABLE COMBUSTION")
print("="*50)

# --- 3. Static Mathematical Plot ---
print("Generating static coherence plot...")
fig_stat = plt.figure(figsize=(10, 5))
fig_stat.patch.set_facecolor('#050510')
ax = fig_stat.add_subplot(111)
ax.set_facecolor('#050510')

ax.plot(t_sim, r_res, color='#ff2a00', lw=2.5, label=f'Destructive Resonance (K > K_c)')
ax.plot(t_sim, r_stab, color='#05d9e8', lw=2.5, label=f'Stable Combustion (K < K_c)')
ax.axhline(y=0.5, color='white', linestyle='--', alpha=0.3, label='Phase Lock Threshold')

ax.set_title('ORDER PARAMETER r(t): Thermoacoustic Phase Transition', color='white', pad=15)
ax.set_xlabel('Simulation Time', color='white')
ax.set_ylabel('Phase Coherence (r)', color='white')
ax.tick_params(colors='white')
ax.legend(facecolor='#050510', labelcolor='white')
for spine in ax.spines.values(): spine.set_edgecolor('#333')

stat_path = SAVE_PATH + 'thermoacoustic_plot.png'
plt.savefig(stat_path, dpi=120, bbox_inches='tight')
plt.close()

# --- 4. Generative Art (GIF) ---
print("Rendering topological phase flow GIF...")
BG_COLOR = '#050510' 
fig_a = plt.figure(figsize=(14, 7))
fig_a.patch.set_facecolor(BG_COLOR)

al = fig_a.add_subplot(1, 2, 1, projection='polar')
ar = fig_a.add_subplot(1, 2, 2, projection='polar')

def draw_frame(frame):
    al.clear(); ar.clear()
    for ax in [al, ar]:
        ax.set_facecolor(BG_COLOR)
        ax.set_xticks([]); ax.set_yticks([])
        for spine in ax.spines.values(): spine.set_visible(False)

    tail_len = 15
    start_idx = max(0, frame - tail_len)
    
    # LEFT: Destructive Resonance (Red)
    for j in range(N):
        phases = sa_res[start_idx:frame+1, j] % 1.0
        rads = np.ones(len(phases)) * 0.8 
        alphas = np.linspace(0.0, 0.9, len(phases))
        for i in range(len(phases)-1):
            al.plot(2*np.pi*phases[i:i+2], rads[i:i+2], color='#ff2a00', alpha=alphas[i], lw=2.5)
        if len(phases) > 0:
            al.scatter(2*np.pi*phases[-1], rads[-1], color='#ffffff', s=40, alpha=1.0)
    al.text(0.5, -0.1, f"DESTRUCTIVE RESONANCE (r = {r_res[frame]:.2f})", transform=al.transAxes, ha='center', color='#ff2a00', fontsize=14, fontweight='bold')

    # RIGHT: Stable Combustion (Cyan)
    for j in range(N):
        phases = sa_stab[start_idx:frame+1, j] % 1.0
        rads = np.linspace(0.6, 1.0, len(phases)) 
        alphas = np.linspace(0.0, 0.9, len(phases))
        for i in range(len(phases)-1):
            ar.plot(2*np.pi*phases[i:i+2], rads[i:i+2], color='#05d9e8', alpha=alphas[i], lw=2.5)
        if len(phases) > 0:
            ar.scatter(2*np.pi*phases[-1], rads[-1], color='#05d9e8', s=30, alpha=1.0)
    ar.text(0.5, -0.1, f"STABLE COMBUSTION (r = {r_stab[frame]:.2f})", transform=ar.transAxes, ha='center', color='#05d9e8', fontsize=14, fontweight='bold')

    fig_a.suptitle('THERMOACOUSTIC PHASE GEOMETRY: ROCKET PROPULSION', color='white', fontsize=18, fontweight='bold', y=0.95)

ani = animation.FuncAnimation(fig_a, draw_frame, frames=NF, interval=60, blit=False)
gif_path = SAVE_PATH + 'thermoacoustic_phase_flow.gif'
ani.save(gif_path, writer='pillow', fps=20, dpi=120, savefig_kwargs={'facecolor': BG_COLOR})

print(f"Done! Saved: {stat_path} and {gif_path}")

if IN_COLAB:
    from google.colab import files
    files.download(stat_path)
    files.download(gif_path)
