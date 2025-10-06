<script>
    import { getContext, untrack } from 'svelte';
    import {
        toISOString, toLocalDate, toViewWithLocalDates, isFunction, task, flushDebounce, hasYScroll
    } from '#lib';

    const { selectedEvent } = $props();// 🆕 Added for side panel

    let {
        datesSet, eventAllUpdated, _auxiliary, _activeRange, _filteredEvents, _scrollable, _bodyEl, _tasks, _recheckScrollable,
        _queue, _view
    } = getContext('state');

    // datesSet callback
    $effect(() => {
        $_activeRange;
        untrack(() => {
            if (isFunction($datesSet)) {
                $datesSet({
                    start: toLocalDate($_activeRange.start),
                    end: toLocalDate($_activeRange.end),
                    startStr: toISOString($_activeRange.start),
                    endStr: toISOString($_activeRange.end),
                    view: toViewWithLocalDates($_view)
                });
            }
        });
    });

    // eventAllUpdated callback
    $effect(() => {
        $_filteredEvents;
        untrack(() => {
            if (isFunction($eventAllUpdated)) {
                task(() => $eventAllUpdated({ view: toViewWithLocalDates($_view) }), 'eau', _tasks);
            }
        });
    });

    $effect(() => {
        $_queue;
        untrack(() => {
            flushDebounce($_queue);
        });
    });

    $effect(() => {
        $_recheckScrollable;
        untrack(() => {
            if ($_bodyEl) {
                $_scrollable = hasYScroll($_bodyEl);
            }
            $_recheckScrollable = false;
        });
    });
</script>

{#each $_auxiliary as Component}
    <Component/>
{/each}

<!-- 🆕 Event Details Side Panel -->
{#if selectedEvent}
    <div class="event-details-panel">
        <div class="event-details-header">
            <h3>{selectedEvent.title}</h3>
            <button on:click={() => selectedEvent = null}>×</button>
        </div>
        <div class="event-details-body">
            <p><strong>Start:</strong> {selectedEvent.start}</p>
            <p><strong>End:</strong> {selectedEvent.end}</p>
            {#if selectedEvent.description}
                <p><strong>Description:</strong> {selectedEvent.description}</p>
            {/if}
        </div>
    </div>
{/if}

<style>
.event-details-panel {
    position: fixed;
    top: 0;
    right: 0;
    width: 340px;
    height: 100%;
    background: #fff;
    box-shadow: -4px 0 8px rgba(0, 0, 0, 0.25);
    padding: 1rem;
    overflow-y: auto;
    z-index: 1000;
    transition: transform 0.3s ease;
}

.event-details-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid #ddd;
    margin-bottom: 1rem;
}

.event-details-header button {
    background: transparent;
    border: none;
    font-size: 1.4rem;
    cursor: pointer;
}

.event-details-body p {
    margin: 0.5rem 0;
}
</style>
