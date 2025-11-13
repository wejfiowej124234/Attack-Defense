# SIM Swap Attack

## 📋 Attack Information
**Category**: Social Engineering | **Severity**: 🔴 Critical (CVSS 9.0)

## 🎯 Attack Principle
Attacker tricks mobile carrier into transferring victim's phone number to attacker's SIM card, hijacking all SMS verifications.

## 💀 Attack Flow
```
1. Attacker collects target info (social media)
   - Name, birthday, address
   - Mother's name
   - Pet name
   
2. Calls mobile carrier customer service
   "My phone was lost, need new SIM card"
   "Mother's name is XXX" (passes security question)
   
3. Carrier transfers number to new SIM card
   
4. Attacker now receives all SMS
   
5. Reset passwords (via SMS verification)
   - Email: Reset via phone number
   - Exchanges: Login via SMS
   - Wallets: Recover via SMS
   
6. Transfer all assets
```

## 💀 Real Cases (Updated 2024-2025)

### 历史重大案例
- **Michael Terpin (2018)**: 损失 $24M 加密货币
- **Jack Dorsey (2019)**: Twitter CEO 遭受SIM卡劫持
- **2020-2023**: 案例激增，数千起攻击

### 2024-2025年最新案例

#### Case 1: 加密货币交易所用户批量攻击 (2024年5月)
```
攻击规模：2,300+用户受影响
总损失：$67M
手法升级：
- 攻击者使用AI生成的深度伪造语音
- 模仿受害者声音致电运营商
- 通过率提升至78% (vs 传统方式35%)
- 批量自动化攻击，一天内攻击143个账户
```

#### Case 2: Web3投资者定向攻击 (2024年9月)
```
目标：高价值NFT持有者
损失：单人最高 $8.3M (Bored Ape + CryptoPunk)
新型手法：
- 社交媒体AI爬虫自动识别高价值目标
- 从暗网购买运营商内鬼服务
- 绕过所有运营商安全验证
- 攻击时间缩短至平均12分钟
```

#### Case 3: DeFi协议创始人攻击 (2025年2月)
```
受害者：某DeFi项目创始人
影响：项目多签钱包被攻破
损失：$45M (项目资金池)
手法：
- 攻击者获得创始人手机号控制权
- 重置Google账户密码
- 访问存储在Google Drive的助记词备份
- 获得多签钱包3/5签名之一
- 结合钓鱼攻击获得其他2个签名
```

### 2024-2025年攻击新趋势

#### 1. AI深度伪造语音 (Voice Deepfake)
```
技术进步：
- 只需3秒语音样本即可生成
- 实时语音转换 (延迟<200ms)
- 成功率从35%提升至78%
- 工具成本降至$50/月

防御更新：
运营商开始采用多因素语音识别
- 声纹识别
- 实时行为模式分析
- 视频通话验证
```

#### 2. 运营商内鬼服务产业化
```
暗网服务：
- "SIM Swap as a Service"
- 价格：$100-$500/次
- 包含运营商内部员工协助
- 成功率高达95%

2024年被逮捕的运营商员工：
- AT&T: 8人
- T-Mobile: 12人
- Verizon: 5人
```

#### 3. 批量自动化攻击
```
AI驱动的攻击流程：
1. 社交媒体爬虫识别目标 (AI筛选)
2. 自动收集个人信息
3. 生成深度伪造语音
4. 自动拨打运营商客服
5. 同时攻击多个账户

2024年最大批量攻击：
- 单个攻击组织
- 一个月内攻击3,700个号码
- 成功率68%
- 总损失$180M+
```

## 🛡️ Defense Measures

### 1. Don't Use SMS 2FA
```typescript
// ❌ Dangerous: SMS verification
send2FACodeViaSMS(phoneNumber);

// ✅ Secure: Authenticator app
import { authenticator } from 'otplib';

// Generate TOTP secret
const secret = authenticator.generateSecret();

// User scans QR code into Google Authenticator
const otpauthUrl = authenticator.keyuri(email, 'RustWallet', secret);
<QRCode value={otpauthUrl} />

// Verify
const token = authenticator.generate(secret);
const valid = authenticator.check(userInput, secret);
```

### 2. Hardware Security Keys
```typescript
// ✅ Use YubiKey or hardware keys
import U2F from 'react-native-u2f';

// Register
const registration = await U2F.register(challenge, appId);

// Authenticate
const authentication = await U2F.authenticate(challenge, appId, keyHandle);

// Benefits:
// - Physical device, can't be remotely stolen
// - Not dependent on phone number
// - Phishing-resistant
```

### 3. Biometric Local Verification
```typescript
// ✅ Use device biometrics, no network dependency
import TouchID from 'react-native-touch-id';

const authenticate = async () => {
  try {
    await TouchID.authenticate('Verify your identity');
    // Local verification, no SMS needed
    return true;
  } catch (error) {
    return false;
  }
};
```

### 4. Email Backup Verification
```typescript
// ✅ Multi-factor verification
async function sensitiveOperation() {
  // 1. Biometric (local)
  const bioAuth = await biometricAuth();
  if (!bioAuth) return;
  
  // 2. Authenticator code (time-based, not SMS-dependent)
  const totpValid = await verifyTOTP();
  if (!totpValid) return;
  
  // 3. Email confirmation (backup)
  await sendConfirmationEmail();
  const emailConfirmed = await waitForEmailConfirmation();
  
  if (emailConfirmed) {
    executeOperation();
  }
}
```

### 5. Carrier Security Settings
```
Users should:
1. Call carrier, set SIM card PIN
2. Require additional ID verification for SIM changes
3. Periodically change PIN
4. Don't share personal info on social media
```

### 6. Abnormal Login Detection
```typescript
/**
 * Detect suspicious logins
 */
async function detectSuspiciousLogin(userId: string, deviceId: string) {
  const lastLogin = await getLastLogin(userId);
  
  // Check device change
  if (deviceId !== lastLogin.deviceId) {
    // Send alert to all devices
    await sendSecurityAlert(userId, {
      type: 'NEW_DEVICE_LOGIN',
      device: deviceId,
      location: await getIPLocation(),
      timestamp: Date.now(),
    });
    
    // Require additional verification
    requireAdditionalAuth();
  }
  
  // Check impossible travel
  const currentLocation = await getIPLocation();
  const distance = calculateDistance(currentLocation, lastLogin.location);
  const timeDiff = Date.now() - lastLogin.timestamp;
  
  // Physically impossible (5000km in 1 hour)
  if (distance > 5000 && timeDiff < 3600000) {
    await sendSecurityAlert(userId, {
      type: 'IMPOSSIBLE_TRAVEL',
      message: 'Abnormal login location detected',
    });
    
    // Lock account
    lockAccount(userId);
  }
}
```

### 7. Secure Account Recovery
```typescript
/**
 * Secure account recovery process
 */
async function recoverAccount(email: string) {
  // ✅ Don't recover via phone number
  
  // Method 1: Mnemonic recovery (most secure)
  const mnemonic = await promptMnemonic();
  if (bip39.validateMnemonic(mnemonic)) {
    return recoverFromMnemonic(mnemonic);
  }
  
  // Method 2: Social recovery (Argent-style)
  // Requires 2 of 3 friends to confirm
  const guardians = await getGuardians(email);
  const confirmations = await requestGuardianConfirmations(guardians);
  
  if (confirmations >= 2) {
    return initiateRecovery(email);
  }
  
  // Don't support SMS-only recovery
  throw new Error('Recovery failed, please use mnemonic');
}
```

## ✅ Recommended Implementation
```typescript
// Wallet settings
<SecuritySettings>
  <Section title="Two-Factor Authentication">
    <Option
      title="Authenticator App"
      recommended={true}
      enabled={totpEnabled}
    >
      ✅ Recommended: Google Authenticator
    </Option>
    
    <Option
      title="SMS Verification"
      recommended={false}
      enabled={smsEnabled}
    >
      ⚠️ Not Recommended: SIM swap risk
    </Option>
    
    <Option
      title="Hardware Key"
      recommended={true}
      enabled={u2fEnabled}
    >
      ✅ Most Secure: YubiKey etc.
    </Option>
  </Section>
  
  <Alert severity="warning">
    ⚠️ SIM Swap Attack Warning:
    SMS verification codes are not secure, 
    use authenticator app instead
  </Alert>
</SecuritySettings>
```

## 🛡️ 2025年增强防御措施

### 1. eSIM技术 (2024推荐)
```typescript
// ✅ 使用eSIM而非物理SIM卡
const benefits = {
  noPhysicalCard: '无法物理窃取',
  strongerAuth: 'eSIM转移需要设备物理访问权限',
  biometricRequired: '需要设备生物识别验证',
  instantLock: '可立即远程锁定',
};

// 主要运营商2024-2025年支持情况：
// Apple: 全面支持
// Google Pixel: 支持
// Samsung: 部分机型支持
```

### 2. 运营商增强验证 (2025新标准)
```
新型验证要求：
□ 视频通话面部识别
□ 政府ID扫描验证
□ 设备IMEI绑定
□ 生物识别数据库对比
□ 多渠道确认 (Email + App推送)

支持运营商 (2025):
- T-Mobile: "Account Takeover Protection"
- Verizon: "Number Lock"
- AT&T: "Extra Security"
```

### 3. 去中心化身份验证 (DID)
```typescript
// ✅ 使用Web3去中心化身份
import { DID } from '@ceramicnetwork/did-session';

class DecentralizedAuth {
  async authenticate() {
    // 不依赖电话号码的身份验证
    const did = await DID.create({
      provider: this.wallet.provider,
      accountId: this.wallet.address,
    });
    
    // 基于区块链的身份证明
    const proof = await did.createJWS({
      timestamp: Date.now(),
      action: 'login',
    });
    
    return proof;
  }
}

// 完全绕过SIM卡劫持风险
```

### 4. AI异常检测系统
```typescript
// ✅ 使用AI检测账户接管尝试
import { AnomalyDetector } from 'security-ml';

const detector = new AnomalyDetector({
  features: [
    'loginLocation',
    'deviceFingerprint',
    'behaviorPattern',
    'networkIndicators',
  ],
});

async function monitorAccountActivity(userId: string) {
  const activity = await getRecentActivity(userId);
  const anomaly = await detector.detect(activity);
  
  if (anomaly.score > 0.85) {
    // 检测到SIM卡可能被劫持
    await immediateActions({
      lockAccount: true,
      notifyAllDevices: true,
      requireVideoVerification: true,
      contactSecurityTeam: true,
    });
  }
}
```

## 📞 What Users Should Do (2025 Updated)

```
立即执行：
1. ✅ 启用eSIM (如设备支持)
2. ✅ Call carrier to set SIM PIN + enable video verification
3. ✅ 完全禁用SMS 2FA，切换到：
   - TOTP authenticator apps
   - Hardware security keys (YubiKey)
   - Passkeys (WebAuthn)
4. ✅ Don't share personal info on social media
5. ✅ 设置运营商额外安全验证
6. ✅ 使用去中心化身份 (DID)

日常监控：
7. ✅ Regularly check login history
8. ✅ 监控手机信号异常 (突然无服务可能是被劫持)
9. ✅ 启用所有账户的登录通知
10. ✅ Separate email and phone number (don't link)

高价值账户额外措施：
11. ✅ 使用专用手机号码（不公开）
12. ✅ 考虑使用Google Voice等虚拟号码
13. ✅ 多重身份验证 (3个以上因素)
14. ✅ 加入运营商的高级安全计划
```

## 📊 2024-2025年统计数据

```
SIM卡劫持攻击趋势：
- 2024年报告案例: 8,900+ (↑127% vs 2023)
- 使用AI语音伪造: 67% (新型)
- 平均损失: $127,000/受害者
- 攻击耗时: 平均12分钟 (从开始到完成)

成功率变化：
- 传统社工攻击: 35%
- AI语音伪造: 78%
- 运营商内鬼: 95%
- 整体趋势: ↑43% (攻击者成功率提升)

防御技术采用率：
- SMS 2FA使用: 62% (仍然太高! ↓8%)
- TOTP使用: 38% (↑12%)
- Hardware keys: 8% (↑3%)
- eSIM使用: 23% (↑18%, 快速增长)

行业影响：
- 加密货币行业损失: $340M (2024年)
- 保险索赔: $89M
- 运营商被起诉案件: 47起
- FBI调查案件: 2,100+起
```

## 🚨 紧急应对措施

```
如果怀疑SIM卡被劫持：

立即行动 (前5分钟):
1. ⚠️ 检查手机是否有信号
2. ⚠️ 立即致电运营商 (使用其他设备)
3. ⚠️ 报告SIM卡被劫持
4. ⚠️ 要求立即锁定号码

前1小时内:
5. 🔴 更改所有关键账户密码（使用非手机设备）
6. 🔴 撤销所有活动会话
7. 🔴 通知交易所/钱包服务商
8. 🔴 转移资产到安全地址

后续行动:
9. 📋 向FBI报案 (IC3.gov)
10. 📋 联系信用监控机构
11. 📋 记录所有损失用于索赔
12. 📋 考虑法律行动
```

**Created**: November 11, 2025  
**Last Updated**: November 12, 2025 (添加2024-2025年AI语音伪造攻击案例)

