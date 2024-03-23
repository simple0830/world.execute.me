name: world.execute.me

on:
  issues:
    types: [opened, edited]

permissions:
    contents: write
    issues: write
    pull-requests: write

jobs:
  EXECUTION:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
        
      - name: Extract Question and Answer from issue body
        id: interaction-process
        env:
          HEART: ${{ secrets.Heart }} # ILLEGAL ARGUMENTS
        run: |
          set +e
          QUESTION=$(echo '${{ github.event.issue.body }}' | grep -oP 'Question:.*$')
          ANSWER=$(echo '${{ github.event.issue.body }}' | grep -vP 'Question:.*$')
          ANSWER=$(eval "$ANSWER" 2>&1) # world.execute(DeBug);
          echo "question=$QUESTION" >> $GITHUB_ENV
          echo "answer=$ANSWER" >> $GITHUB_ENV

      - name: Comment on issue with response
        id: comment
        uses: actions-cool/issues-helper@v1
        with:
          actions: 'create-comment'
          token: ${{ secrets.GITHUB_TOKEN }}
          issue-number: ${{ github.event.issue.number }}
          body: '`Your question was: ${{ env.question }} Your answer is: ${{ env.answer }}`'

      - name: sleep
        run: sleep 3

      - name: Delete comment
        uses: actions-cool/issues-helper@v1
        with:
          actions: 'delete-comment'
          token: ${{ secrets.GITHUB_TOKEN }}
          comment-id: ${{ steps.comment.outputs.comment-id }} 

      - name: Close Issue
        uses: actions/github-script@v6
        with:
          github-token: ${{secrets.GITHUB_TOKEN}}
          script: |
            github.rest.issues.update({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: ${{ github.event.issue.number }},
              body: 'This task has been completed. 🎉',
              state: 'closed', 
              title: 'Task - Completed'
            });

      - name: Core_Call
        run: |
          curl -L \
            -X POST \
            -H "Accept: application/vnd.github+json" \
            -H "Authorization: Bearer ${{ secrets.CORESECRET }}" \
            -H "X-GitHub-Api-Version: 2022-11-28" \
            -H "User-Agent: Webhook" \
            https://api.github.com/repos/ProbiusOfficial/Core/dispatches \
            -d '{"event_type":"Core_Call","client_payload":{"unit":false,"integration":true}}' 
