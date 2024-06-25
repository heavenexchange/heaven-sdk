# Heaven

## Usage
```bash
npm install heaven-sdk
```

### Create a new pool ([example](./examples/simple/create-new-pool.ts))

```typescript
/* eslint-disable max-len */
import {
    ComputeBudgetProgram,
    Connection,
    Keypair,
    PublicKey,
    Transaction,
    sendAndConfirmTransaction,
} from '@solana/web3.js';
import { BN } from 'bn.js';
import { Heaven } from 'heaven-sdk';

export async function createLiquidityPoolExample() {
    const owner = Keypair.generate();
    const connection = new Connection(
        'https://api.devnet.solana.com/', // Replace with your preferred Solana RPC endpoint
        'confirmed'
    );

    // Initialize a new liquidity pool
    const pool = await Heaven.init({
        base: new PublicKey('EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v'), // USDC;
        quote: new PublicKey('So11111111111111111111111111111111111111112'), // WSOL;
        connection: connection,
        owner: owner.publicKey,
        network: 'devnet',
    });

    const ix = await pool.createIx({
        // amount of base token to deposit
        baseAmount: new BN(1000_000 * 10 ** pool.baseTokenMintDecimals),
        // amount of quote token to deposit
        quoteAmount: new BN(1000 * 10 ** pool.quoteTokenMintDecimals),
        // sellTax BPS = 100 / 10000 * 100 = 1%;
        sellTax: new BN(100),
        // buyTax BPS = 25 / 10000 * 100 = 0.25%;
        buyTax: new BN(25),
        // locking liquidity
        lp: 'lock', // or 'burn' to burn LP tokens
        // Lock liquidity for 60 seconds
        lockLiquidityUntil: new Date(new Date().getTime() + 60 * 1000),
        // Open pool 60 seconds after creation
        openPoolAt: new Date(new Date().getTime() + 60 * 1000),
        event: '',
    });

    await sendAndConfirmTransaction(
        connection,
        new Transaction().add(
            ComputeBudgetProgram.setComputeUnitLimit({
                units: 300000,
            }),
            ix
        ),
        [owner],
        {
            commitment: 'confirmed',
        }
    );
}
```

### Swap exact in ([example](./examples/simple/swap-exact-in.ts))

```typescript
/* eslint-disable max-len */
/* eslint-disable no-mixed-spaces-and-tabs */
import {
    Connection,
    Keypair,
    PublicKey,
    Transaction,
    sendAndConfirmTransaction,
} from '@solana/web3.js';
import { BN } from 'bn.js';
import { Heaven } from 'heaven-sdk';

export async function swapExactInExample() {
    const connection = new Connection(
        'https://api.devnet.solana.com',
        'confirmed'
    );
    const liquidityPoolAddress = new PublicKey('...'); // Insert the liquidity pool address
    const user = Keypair.generate();

    // Load an existing pool by id
    const pool = await Heaven.load({
        id: liquidityPoolAddress,
        network: 'devnet',
        user: user.publicKey,
        connection,
    });

    // Swapping in 1000 base tokens for quote tokens
    const amount = new BN(1000 * 10 ** pool.baseTokenMintDecimals);

    const slippage = new BN(100); // 1%

    // Quote the minimum amount of quote tokens that will be received
    // based on the provided slippage
    const quoteResult = await pool.quoteSwapIn({
        amount,
        inputSide: 'base',
        slippage,
    });

    const ix = await pool.swapInIx({
        amount,
        quoteResult,
        event: '',
    });

    await sendAndConfirmTransaction(
        connection,
        new Transaction().add(ix),
        [user],
        {
            commitment: 'confirmed',
        }
    );
}
```

### Swap exact out ([example](./examples/simple/swap-exact-out.ts))

```typescript
/* eslint-disable max-len */
/* eslint-disable no-mixed-spaces-and-tabs */
import {
    Connection,
    Keypair,
    PublicKey,
    Transaction,
    sendAndConfirmTransaction,
} from '@solana/web3.js';
import { BN } from 'bn.js';
import { Heaven } from 'heaven-sdk';

export async function swapExactOutExample() {
    const connection = new Connection(
        'https://api.devnet.solana.com',
        'confirmed'
    );
    const liquidityPoolAddress = new PublicKey('...'); // Insert the liquidity pool address
    const user = Keypair.generate();

    // Load the pool
    const pool = await Heaven.load({
        id: liquidityPoolAddress,
        network: 'devnet',
        user: user.publicKey,
        connection,
    });

    // Swapping out 1000 base tokens using quote tokens
    const amountOut = new BN(1000 * 10 ** pool.baseTokenMintDecimals);

    const slippage = new BN(100); // 1%

    // Quote the maximum amount of quote tokens that will be spent
    // based on the provided slippage
    const quoteResult = await pool.quoteSwapOut({
        amount: amountOut,
        inputSide: 'quote',
        slippage,
    });

    const ix = await pool.swapOutIx({
        amount: amountOut,
        quoteResult,
        event: '',
    });

    await sendAndConfirmTransaction(
        connection,
        new Transaction().add(ix),
        [user],
        {
            commitment: 'confirmed',
        }
    );
}
```

### Add liquidity ([example](./examples/simple/add-liquidity.ts))

```typescript
/* eslint-disable max-len */
/* eslint-disable no-mixed-spaces-and-tabs */
import {
    Connection,
    Keypair,
    PublicKey,
    Transaction,
    sendAndConfirmTransaction,
} from '@solana/web3.js';
import { BN } from 'bn.js';
import { Heaven } from 'heaven-sdk';

export async function addLpExample() {
    const connection = new Connection(
        'https://api.devnet.solana.com',
        'confirmed'
    );
    const liquidityPoolAddress = new PublicKey('...'); // Insert the liquidity pool address
    const user = Keypair.generate();

    // Load the pool
    const pool = await Heaven.load({
        id: liquidityPoolAddress,
        network: 'devnet',
        user: user.publicKey,
        connection,
    });

    // Adding 1000 base tokens
    const baseAmount = new BN(1000 * 10 ** pool.baseTokenMintDecimals);

    // Calculate the maximum amount of quote tokens that needs to be added as well
    // based on the provided slippage
    const quoteResult = await pool.quoteAddLp({
        inputSide: 'base',
        amount: baseAmount,
        slippage: new BN(100), // 1%
    });

    const ix = await pool.addLpIx({
        quoteResult,
        event: '',
    });

    await sendAndConfirmTransaction(
        connection,
        new Transaction().add(ix),
        [user],
        {
            commitment: 'confirmed',
        }
    );
}
```

### Remove liquidity ([example](./examples/simple/remove-liquidity.ts))

```typescript
/* eslint-disable max-len */
/* eslint-disable no-mixed-spaces-and-tabs */
import {
    Connection,
    Keypair,
    PublicKey,
    Transaction,
    sendAndConfirmTransaction,
} from '@solana/web3.js';
import { BN } from 'bn.js';
import { Heaven } from 'src';

export async function removeLpExample() {
    const connection = new Connection(
        'https://api.devnet.solana.com',
        'confirmed'
    );
    const liquidityPoolAddress = new PublicKey('...'); // Insert the liquidity pool address
    const user = Keypair.generate();

    // Load the pool
    const pool = await Heaven.load({
        id: liquidityPoolAddress,
        network: 'devnet',
        user: user.publicKey,
        connection,
    });

    // Remove 1000 lp tokens
    const lpAmount = new BN(1000 * 10 ** pool.lpTokenMintDecimals);

    // Calculate the minimum amount of base and quote tokens that will be received
    // based on the provided slippage
    const quoteResult = await pool.quoteRemoveLp({
        amount: lpAmount,
        slippage: new BN(100), // 1%
    });

    const ix = await pool.removeLpIx({
        quoteResult,
        event: '',
    });

    await sendAndConfirmTransaction(
        connection,
        new Transaction().add(ix),
        [user],
        {
            commitment: 'confirmed',
        }
    );
}
```

### Claim Swap Tax ([example](./examples/simple/claim-tax.ts))

```typescript
/* eslint-disable max-len */
/* eslint-disable no-mixed-spaces-and-tabs */
import {
    Connection,
    Keypair,
    PublicKey,
    Transaction,
    sendAndConfirmTransaction,
} from '@solana/web3.js';
import { Heaven } from 'heaven-sdk';

export async function claimTaxExample() {
    const connection = new Connection(
        'https://api.devnet.solana.com',
        'confirmed'
    );
    const liquidityPoolAddress = new PublicKey('...'); // Insert the liquidity pool address
    const user = Keypair.generate();

    // Load the pool
    const pool = await Heaven.load({
        id: liquidityPoolAddress,
        network: 'devnet',
        user: user.publicKey,
        connection,
    });

    // Get the current tax balances
    const baseAmount = await pool.baseTaxBalance;
    const quoteAmount = await pool.quoteTaxBalance;

    // Claim all of the tax
    const ix = await pool.claimTaxIx({
        base: baseAmount,
        quote: quoteAmount,
        event: '',
    });

    await sendAndConfirmTransaction(
        connection,
        new Transaction().add(ix),
        [user],
        {
            commitment: 'confirmed',
        }
    );
}
```

### Claim Lp Tokens ([example](./examples/simple/claim-lp-tokens.ts))

```typescript
/* eslint-disable max-len */
/* eslint-disable no-mixed-spaces-and-tabs */
import {
    Connection,
    Keypair,
    PublicKey,
    Transaction,
    sendAndConfirmTransaction,
} from '@solana/web3.js';
import { Heaven } from 'heaven-sdk';

export async function claimLpTokensExample() {
    const connection = new Connection(
        'https://api.devnet.solana.com',
        'confirmed'
    );
    const liquidityPoolAddress = new PublicKey('...'); // Insert the liquidity pool address
    const user = Keypair.generate();

    // Load the pool
    const pool = await Heaven.load({
        id: liquidityPoolAddress,
        network: 'devnet',
        user: user.publicKey,
        connection,
    });

    // Get the current locked lp token account balance
    const amount = await pool.lockedLpTokenBalance;

    // Claim all of the locked lp tokens
    const ix = await pool.claimLpTokensIx({
        amount,
    });

    await sendAndConfirmTransaction(
        connection,
        new Transaction().add(ix),
        [user],
        {
            commitment: 'confirmed',
        }
    );
}
```

### Update Liquidity Pool ([example](./examples/simple/update-liquidity-pool.ts))

```typescript
/* eslint-disable max-len */
/* eslint-disable no-mixed-spaces-and-tabs */
import {
    Connection,
    Keypair,
    PublicKey,
    Transaction,
    sendAndConfirmTransaction,
} from '@solana/web3.js';
import { Heaven } from 'heaven-sdk';

export async function updateLiquidityPoolExample() {
    const connection = new Connection(
        'https://api.devnet.solana.com',
        'confirmed'
    );
    const liquidityPoolAddress = new PublicKey('...'); // Insert the liquidity pool address
    const user = Keypair.generate();

    // Load the pool
    const pool = await Heaven.load({
        id: liquidityPoolAddress,
        network: 'devnet',
        user: user.publicKey,
        connection,
    });

    const enableAddLpIx = await pool.enableAddLpIx();
    const enableRemoveLpIx = await pool.enableRemoveLpIx();
    const enableSwapIx = await pool.enableSwapIx();
    const disableAddLpIx = await pool.disableAddLpIx();
    const disableRemoveLpIx = await pool.disableRemoveLpIx();
    const disableSwapIx = await pool.disableSwapIx();

    const currentLpLockTs = await pool.getCurrentLpLockTimestamp();
    const extendedLpLockTs = currentLpLockTs.getTime() + 60 * 60 * 1000; // extend the lock by 1 hour

    // Note: you can only extend lp lock, not shorten it
    const extendLpLockIx = await pool.extendLpLockIx({
        lockLiquidityUntil: new Date(extendedLpLockTs),
    });

    // Note: you can only update the open pool at timestamp if the previous open pool at timestamp has not passed
    const updateOpenPoolAtIx = await pool.updateOpenPoolAtIx({
        openPoolAt: new Date(Date.now() + 60 * 60 * 1000), // open the pool in 1 hour from now
    });

    await sendAndConfirmTransaction(
        connection,
        new Transaction().add(
            enableAddLpIx,
            enableRemoveLpIx,
            enableSwapIx,
            disableAddLpIx,
            disableRemoveLpIx,
            disableSwapIx,
            extendLpLockIx,
            updateOpenPoolAtIx
        ),
        [user],
        {
            commitment: 'confirmed',
        }
    );
}
```