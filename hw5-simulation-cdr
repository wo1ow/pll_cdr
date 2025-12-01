%% 10Gbps CDR 行为级仿真 (匹配作业图片参数)
clear; clc; close all;

%% 1. 参数定义 (完全依据图片)
BitRate = 10e9;             % 10 Gbps (UI = 100ps)
UI = 1/BitRate;
Fs = 100e9;                % 采样率 1THz (1ps time step)，保证高精度
dt = 1/Fs;

% 环路滤波器参数 (图片中的 R 和 C)
R = 200;                    % 200 Ohm
C = 200e-12;                % 200 pF

% CP 和 VCO 参数
Icp = 300e-6;               % 300 uA
Kvco = 1e9;                 % 1 GHz/V
Kpd = 1/pi;                 % 图片标注的鉴相增益

% 抖动参数 (作业要求: 1MHz, 0.2UI)
Jitter_Freq = 1e6;          
Jitter_UI = 0.2;
Jitter_Amp = Jitter_UI * UI; % 峰峰值对应的时间长度


%% 2. 信号生成 (发送端 Tx) - 采用纯 MATLAB LFSR 生成 PRBS-7

NumBits = 20000;            % 仿真 20000 个 bit (保持不变)
t = (0:dt:(NumBits*UI)-dt)';
NumSteps = length(t);

% --- 新的 PRBS-7 序列生成逻辑 (不依赖 Toolbox) ---
% 1. PRBS 参数设置
N = 7;                            % 阶数 N=7
LFSR_State = [1 0 0 0 0 0 0];     % 初始状态（不能全为 0）
Tx_Bits = zeros(NumBits, 1);      % 预分配输出数组

% 2. 循环生成序列
for i = 1:NumBits
    % 多项式 x^7 + x^6 + 1 -> 反馈位 = LFSR_State(7) XOR LFSR_State(6)
    feedback_bit = xor(LFSR_State(7), LFSR_State(6));
    
    % 输出当前 LFSR_State 的最后一位（或者第一位，取决于约定）作为数据
    Tx_Bits(i) = LFSR_State(N); 
    
    % 移位：将反馈位移入第一位
    LFSR_State = [feedback_bit, LFSR_State(1:N-1)];
end

% ----------------------------------------------------

SamplesPerBit = round(UI/dt);

% -- 生成理想的发送时钟 (Tx Clock) (保持不变) --
Tx_Clock_Ideal = 0.5 * sin(2*pi*BitRate*t) + 0.5; 

% -- 生成带抖动的发送数据 (Tx Data) (保持不变) --
Jitter_Phase = (Jitter_Amp/2) * sin(2*pi*Jitter_Freq*t); 
Data_Ideal = rectpulse(Tx_Bits, SamplesPerBit);
Data_Ideal = Data_Ideal(1:NumSteps);
Tx_Data = interp1(t, double(Data_Ideal), t - Jitter_Phase, 'linear', 0.5);
%% 3. CDR 环路仿真 (接收端 Rx)
% 初始化
Vc = 0;                     % 电容积分电压
V_ctrl = 0;                 % VCO 控制电压
Phi_VCO = 0;                % VCO 当前相位
Recovered_Clock = zeros(size(t));

% 预计算输入数据的相位 (用于鉴相)
% 这里简化模型：输入相位 = 理想线性相位 + 抖动
Phi_In = 2*pi*BitRate*t + (2*pi * Jitter_Phase / UI);

for k = 2:NumSteps
    % --- A. VCO 行为 ---
    % 瞬时频率 = 中心频率 + K_vco * V_ctrl
    f_inst = BitRate + Kvco * V_ctrl;
    % 积分得到相位
    Phi_VCO = Phi_VCO + f_inst * dt * 2 * pi;
    % 生成恢复时钟波形 (用于画眼图)
    Recovered_Clock(k) = (sin(Phi_VCO) > 0); 
    
    % --- B. PD (鉴相器) ---
    % 线性化鉴相模型：相位误差 = 输入相位 - 反馈相位
    Phi_Err = Phi_In(k) - Phi_VCO;
    % 归一化到 [-pi, pi]
    Phi_Err = mod(Phi_Err + pi, 2*pi) - pi;
    
    % --- C. CP (电荷泵) ---
    % I_out = Icp * Kpd * Phi_Err
    % 图片中 Kpd = 1/pi, 所以 I_out = Icp * (Phi_Err / pi)
    I_pump = Icp * (Phi_Err / pi);
    
    % --- D. Loop Filter (Z(s) = R + 1/sC) ---
    % 电流流经 R -> 产生 I*R 电压
    % 电流流入 C -> 产生 (1/C)*integral(I) 电压
    Vc = Vc + (I_pump / C) * dt;  % 积分项
    V_ctrl = Vc + I_pump * R;     % 比例项 + 积分项
end

%% 4. 绘图 (完成作业要求的3张图)
figure('Position', [100, 100, 1000, 800]);

% 定义眼图绘制参数
Eye_Period = 2 * UI; 
Fold_Steps = round(Eye_Period/dt);
Num_Traces = 500;
Start_Idx = 10000; % 跳过初始锁定过程

% --- 子图1: 发送数据眼图 (Tx Data Eye) ---
subplot(3,1,1); hold on; grid on;
title('1. 发送数据眼图 (Tx Data Eye, with Jitter)');
for i = 1:Num_Traces
    idx = Start_Idx + (i-1)*SamplesPerBit*2;
    if idx + Fold_Steps > NumSteps, break; end
    plot(t(1:Fold_Steps)*1e12, Tx_Data(idx:idx+Fold_Steps-1), 'b', 'Color', [0 0 1 0.1]);
end
xlabel('Time (ps)'); ylabel('Amp');

% --- 子图2: 发送时钟眼图 (Tx Clock Eye) ---
% 作业里提到的“发送时钟”通常指生成数据的参考时钟
subplot(3,1,2); hold on; grid on;
title('2. 发送时钟 (Tx Reference Clock)');
% 画一段波形即可，时钟是非常干净的
plot(t(1:500)*1e12, Tx_Clock_Ideal(1:500), 'k', 'LineWidth', 1.5);
xlabel('Time (ps)'); ylabel('Amp');
xlim([0 500*dt*1e12]);

% --- 子图3: 恢复时钟眼图 (Recovered Clock Eye) ---
subplot(3,1,3); hold on; grid on;
title('3. 恢复时钟眼图 (Recovered Clock Eye)');
for i = 1:Num_Traces
    idx = Start_Idx + (i-1)*SamplesPerBit*2;
    if idx + Fold_Steps > NumSteps, break; end
    % 这里的恢复时钟应该跟随了抖动
    plot(t(1:Fold_Steps)*1e12, Recovered_Clock(idx:idx+Fold_Steps-1), 'r', 'Color', [1 0 0 0.1]);
end
xlabel('Time (ps)'); ylabel('Amp');
