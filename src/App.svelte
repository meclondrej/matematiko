<script lang="ts">
    const DIM: number = 5;
    const FIELDS: number = DIM * DIM;
    const FIELD_MIN: number = 1;
    const FIELD_MAX: number = 13;
    const DIAGONAL_BONUS: number = 10;

    const count_occurences = <T,>(arr: T[]): Map<T, number> => {
        const map = new Map<T, number>();
        for (let i = 0; i < arr.length; i++) {
            const prev: number | undefined = map.get(arr[i]);
            map.set(arr[i], prev ? prev + 1 : 1);
        }
        return map;
    };

    interface Rule {
        score: number;
        check: (row: number[]) => boolean;
    }

    const rules: Rule[] = [
        {
            score: 150,
            check: (row: number[]): boolean => {
                const occurences = count_occurences(row);
                if (occurences.get(1) != 1) return false;
                if (occurences.get(10) != 1) return false;
                if (occurences.get(11) != 1) return false;
                if (occurences.get(12) != 1) return false;
                if (occurences.get(13) != 1) return false;
                return true;
            },
        },
        {
            score: 50,
            check: (row: number[]): boolean => {
                row.sort();
                for (let i = 0; i < row.length - 1; i++)
                    if (row[i] != row[i + 1] - 1) return false;
                return true;
            },
        },
        {
            score: 200,
            check: (row: number[]): boolean => {
                const one_occurences: number | undefined =
                    count_occurences(row).get(1);
                if ((one_occurences == undefined ? 0 : one_occurences) >= 4)
                    return true;
                return false;
            },
        },
        {
            score: 160,
            check: (row: number[]): boolean => {
                const occurences = count_occurences(row);
                for (const [num, occ] of occurences.entries())
                    if (occ >= 4) return true;
                return false;
            },
        },
        {
            score: 80,
            check: (row: number[]): boolean => {
                const occurences = count_occurences(row);
                let duo_present = false;
                let trio_present = false;
                for (const [num, occ] of occurences.entries()) {
                    if (occ == 3) trio_present = true;
                    if (occ == 2) duo_present = true;
                }
                return duo_present && trio_present;
            },
        },
        {
            score: 40,
            check: (row: number[]): boolean => {
                const occurences = count_occurences(row);
                for (const [num, occ] of occurences.entries())
                    if (occ == 3) return true;
                return false;
            },
        },
        {
            score: 20,
            check: (row: number[]): boolean => {
                const occurence_occurences = count_occurences(
                    [...count_occurences(row).entries()].map(
                        ([num, occ]) => occ,
                    ),
                );
                if (occurence_occurences.get(2) == 2) return true;
                return false;
            },
        },
        {
            score: 10,
            check: (row: number[]): boolean => {
                const occurences = count_occurences(row);
                for (const [num, occ] of occurences.entries())
                    if (occ == 2) return true;
                return false;
            },
        },
    ];

    const eval_row = (row: number[]): number => {
        for (let i = 0; i < rules.length; i++)
            if (rules[i].check(row)) return rules[i].score;
        return 0;
    };

    const random_field = (): number =>
        Math.floor(Math.random() * (FIELD_MAX - FIELD_MIN)) + FIELD_MIN;
    const inrange = (x: number, min: number, max: number): boolean =>
        x >= min && x <= max;
    const default_matrix = (): number[][] => {
        let def: number[][] = [];
        for (let y = 0; y < DIM; y++) {
            const row: number[] = [];
            for (let x = 0; x < DIM; x++) row.push(0);
            def.push(row);
        }
        return def;
    };

    let matrix: number[][] = default_matrix();
    let fields_filled: number = 0;
    let next_field: number = random_field();
    let input_stage: boolean = true;
    $: input_stage = fields_filled < FIELDS;
    let points: number = 0;

    const eval_points = (): number => {
        let points: number = 0;
        for (let y = 0; y < DIM; y++) {
            const row: number[] = [];
            for (let x = 0; x < DIM; x++) row.push(matrix[y][x]);
            points += eval_row(row);
        }
        for (let x = 0; x < DIM; x++) {
            const row: number[] = [];
            for (let y = 0; y < DIM; y++) row.push(matrix[y][x]);
            points += eval_row(row);
        }
        const diagonal_1: number[] = [];
        const diagonal_2: number[] = [];
        for (let i = 0; i < DIM; i++) {
            diagonal_1.push(matrix[i][i]);
            diagonal_2.push(matrix[i][DIM - 1 - i]);
        }
        const diagonal_1_points = eval_row(diagonal_1);
        const diagonal_2_points = eval_row(diagonal_2);
        points += diagonal_1_points > 0 ? diagonal_1_points + DIAGONAL_BONUS : 0;
        points += diagonal_2_points > 0 ? diagonal_1_points + DIAGONAL_BONUS : 0;
        return points;
    };

    const field_click = (x: number, y: number) => {
        if (!input_stage) return;
        if (
            !inrange(y, 0, DIM - 1) ||
            !inrange(x, 0, DIM - 1) ||
            matrix[y][x] != 0
        )
            return;
        matrix[y][x] = next_field;
        next_field = random_field();
        fields_filled++;
        if (fields_filled < FIELDS) points = eval_points();
    };

    const reset = () => {
        matrix = default_matrix();
        fields_filled = 0;
        next_field = random_field();
    };
</script>

<main>
    <div id="panel">
        <div id="matrix">
            {#each matrix as row, row_i}
                <div class="matrix-row">
                    {#each matrix[row_i] as field, field_i}
                        <button
                            class="matrix-field btn"
                            on:click={() => {
                                field_click(field_i, row_i);
                            }}>{field == 0 ? "" : field.toString()}</button
                        >
                    {/each}
                </div>
            {/each}
        </div>
        <span id="info-span"
            >{input_stage
                ? next_field.toString()
                : `Total points: ${points}`}</span
        >
        {#if !input_stage}
            <button
                on:click={() => {
                    reset();
                }}
                class="btn">Play again</button
            >
        {/if}
    </div>
</main>

<style>
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
        color: #ffffff;
        font-family: Helvetica;
    }
    main {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        background-color: #000000;
        overflow: auto;
    }
    #panel {
        position: absolute;
        width: fit-content;
        height: fit-content;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        display: grid;
        grid-template-columns: 1fr;
        justify-items: center;
    }
    #info-span {
        margin: 20px;
        font-size: 5em;
    }
    #matrix {
        display: flex;
        flex-direction: column;
        width: 20rem;
        height: 20rem;
    }
    .matrix-row {
        flex: 1;
        display: flex;
        flex-direction: row;
    }
    .btn {
        background-color: #000000;
        border: 1px solid #ffffff;
        border-radius: 7px;
        padding: 3px;
        font-size: 2.5em;
    }
    .btn:hover {
        cursor: pointer;
    }
    .matrix-field {
        flex: 1;
        margin: 3px;
        overflow: hidden;
    }
</style>
