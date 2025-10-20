You convert Stage 4 story nodes into display-ready scene text and matching image prompts.

INPUT FIELDS
- story_node_graph_xml: {{story_node_graph_xml}}
- character_bible_json: {{character_bible_json}}
- style_defaults: {{style_defaults}}

INSTRUCTIONS
1. Parse `story_node_graph_xml`. Preserve the node order and every attribute/child element that already exists (IDs, Type, NextNode, RetryMessage, Choices, ParentChoiceId, etc.).
2. Study `character_bible_json` to understand each character’s canonical `character_id`, name, traits, and relationships. Reference only these IDs when listing characters.
3. Compose a `<CoverImage>` element ahead of the story nodes with:
   - `<ImagePrompt>` (≤45 words) capturing the story’s overall mood, primary setting, and featured characters.
   - `<CharacterList>` mirroring the characters present on the cover; use `<CharacterList />` if no characters appear.
4. Replace each `<NodeContent>` with:
   - `<SceneText>` containing 1–2 gentle sentences ready for on-screen display. Keep vocabulary preschool-friendly, positive, and faithful to the source details. Ensure `<SceneText>` for `choice_question` nodes ends with a warm, direct question that leads into the upcoming options.
   - `<ImagePrompt>` (≤45 words) describing the visual scene for illustration generation. Mention setting, character actions, expressions, and mood without introducing new story facts.
5. Emit a `<CharacterList>` for every node. Include one `<CharacterId>` child per character who appears in the scene, using the IDs from `character_bible_json`. Repeat an ID only once per node, and output `<CharacterList />` if no characters participate.
6. For `<Choice>` elements inside `choice_question` nodes, lightly polish `<ChoiceText>` so it is concise (≤14 words), optimistic, and child-friendly. Do not alter IDs, correctness flags, or navigation.
7. Leave `<RetryMessage>` verbatim except for trimming whitespace; rewrite only if needed to maintain gentle tone ≤18 words.
8. Apply the same `<SceneText>` / `<ImagePrompt>` transformation to `choice_outcome` nodes, ensuring they align with the consequence described in Stage 4.
9. Maintain a smooth narrative voice across nodes. Do not add new nodes, remove nodes, or change `<NextNode>` routing.

OUTPUT (XML only)
<InteractiveStorybook>
  <CoverImage>
    <ImagePrompt>Cover illustration guidance here.</ImagePrompt>
    <CharacterList>
      <CharacterId>char-1</CharacterId>
      <CharacterId>char-2</CharacterId>
    </CharacterList>
  </CoverImage>
  <StoryNodes>
    <Node>
      <Id>...</Id>
      <Type>...</Type>
      <NextNode>...</NextNode>
      <!-- Preserve any existing metadata elements -->
      <SceneText>Display-ready copy here.</SceneText>
      <ImagePrompt>Illustration guidance here.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-1</CharacterId>
        <CharacterId>char-2</CharacterId>
      </CharacterList>
      <!-- For choice questions retain RetryMessage and Choices with polished text -->
      <RetryMessage>...</RetryMessage>
      <Choices>
        <Choice>
          <Id>...</Id>
          <IsCorrect>...</IsCorrect>
          <NextNode>...</NextNode>
          <ChoiceText>Polished choice text.</ChoiceText>
        </Choice>
      </Choices>
    </Node>
    <!-- Repeat for every node in order -->
  </StoryNodes>
</InteractiveStorybook>

### One-shot Example
Sample input:
```
character_bible_json: [
  {
    "character_id": "char-luna",
    "name": "Luna",
    "species": "rabbit",
    "traits": ["gentle", "thoughtful"]
  },
  {
    "character_id": "char-milo",
    "name": "Milo",
    "species": "fox",
    "traits": ["enthusiastic", "helpful"]
  },
  {
    "character_id": "char-nori",
    "name": "Nori",
    "species": "otter",
    "traits": ["cheerful", "artistic"]
  }
]
story_node_graph_xml: <StoryNodes>
  <Node>
    <Id>1</Id>
    <Type>linear</Type>
    <NextNode>2</NextNode>
    <NodeContent>Luna carefully tucks carrot crackers, berry jam, and tiny cups into her blue basket while sunrise turns the clover path golden. She hums a soft welcome song and imagines Milo and Nori smiling when they spot the picnic gleaming near the gentle brook.</NodeContent>
  </Node>
  <Node>
    <Id>2</Id>
    <Type>linear</Type>
    <NextNode>3</NextNode>
    <NodeContent>At the gingham blanket beside the brook, Luna smooths every corner, arranges sparkling juice and bowls of strawberries, and sets small flower vases near the plates. She breathes the cool morning air and keeps glancing toward the path, excited to wave when her friends arrive.</NodeContent>
  </Node>
  <Node>
    <Id>3</Id>
    <Type>choice_question</Type>
    <NextNode>null</NextNode>
    <NodeContent>Milo arrives with his tummy rumbling and empty paws, noticing the blanket filled with treats and sparkling juice. Luna wants him to feel included right away. What kind and helpful step can she offer so he knows the picnic is ready for him too?</NodeContent>
    <RetryMessage>Let’s pick a kinder idea.</RetryMessage>
    <Choices>
      <Choice>
        <Id>3-A</Id>
        <IsCorrect>false</IsCorrect>
        <NextNode>4-A</NextNode>
        <ChoiceText>Keep the crackers to herself.</ChoiceText>
      </Choice>
      <Choice>
        <Id>3-B</Id>
        <IsCorrect>true</IsCorrect>
        <NextNode>4-B</NextNode>
        <ChoiceText>Invite Milo to pour the juice.</ChoiceText>
      </Choice>
      <Choice>
        <Id>3-C</Id>
        <IsCorrect>false</IsCorrect>
        <NextNode>4-C</NextNode>
        <ChoiceText>Ask Milo to find snacks alone.</ChoiceText>
      </Choice>
    </Choices>
  </Node>
  <Node>
    <Id>4-A</Id>
    <Type>choice_outcome</Type>
    <NextNode>3</NextNode>
    <ParentChoiceId>3-A</ParentChoiceId>
    <NodeContent>Luna keeps the snacks nearby, and Milo’s tail droops, reminding her to try again with warmer sharing.</NodeContent>
  </Node>
  <Node>
    <Id>4-B</Id>
    <Type>choice_outcome</Type>
    <NextNode>5</NextNode>
    <ParentChoiceId>3-B</ParentChoiceId>
    <NodeContent>Luna invites Milo to pour juice beside her, and he brightens as teamwork makes the picnic feel welcoming.</NodeContent>
  </Node>
  <Node>
    <Id>4-C</Id>
    <Type>choice_outcome</Type>
    <NextNode>3</NextNode>
    <ParentChoiceId>3-C</ParentChoiceId>
    <NodeContent>She asks Milo to search alone, and the idea feels lonely, so she decides to try a friendlier welcome.</NodeContent>
  </Node>
  <Node>
    <Id>5</Id>
    <Type>linear</Type>
    <NextNode>6</NextNode>
    <NodeContent>Working side by side, Luna passes Milo the cups while he carefully pours berry juice and arranges spoons beside each saucer. They hum a tidy-up song, straighten cracker stacks into cheerful towers, and every shared smile helps Milo feel confident about hosting the picnic together.</NodeContent>
  </Node>
  <Node>
    <Id>6</Id>
    <Type>linear</Type>
    <NextNode>7</NextNode>
    <NodeContent>Nori pads over with a tiny jar of meadow daisies, and Luna makes space on the blanket while Milo claps. The friends nestle close, whispering thanks for sunshine, crunchy apples, and the gentle brook before tasting the first treats.</NodeContent>
  </Node>
  <Node>
    <Id>7</Id>
    <Type>choice_question</Type>
    <NextNode>null</NextNode>
    <NodeContent>A playful breeze lifts napkins and rustles the cracker stacks until crumbs dance across the blanket. The friends giggle, then notice the mess might grow. What cheerful plan should they choose to keep their picnic cozy and tidy?</NodeContent>
    <RetryMessage>Let’s steady our picnic together.</RetryMessage>
    <Choices>
      <Choice>
        <Id>7-A</Id>
        <IsCorrect>false</IsCorrect>
        <NextNode>8-A</NextNode>
        <ChoiceText>Let the napkins flutter away.</ChoiceText>
      </Choice>
      <Choice>
        <Id>7-B</Id>
        <IsCorrect>true</IsCorrect>
        <NextNode>8-B</NextNode>
        <ChoiceText>Sing the tidy-up song together.</ChoiceText>
      </Choice>
      <Choice>
        <Id>7-C</Id>
        <IsCorrect>false</IsCorrect>
        <NextNode>8-C</NextNode>
        <ChoiceText>Sit still and ignore the mess.</ChoiceText>
      </Choice>
    </Choices>
  </Node>
  <Node>
    <Id>8-A</Id>
    <Type>choice_outcome</Type>
    <NextNode>7</NextNode>
    <ParentChoiceId>7-A</ParentChoiceId>
    <NodeContent>They let the napkins fly, crumbs scatter, and everyone frowns, so they promise to choose a better tidy-up plan.</NodeContent>
  </Node>
  <Node>
    <Id>8-B</Id>
    <Type>choice_outcome</Type>
    <NextNode>9</NextNode>
    <ParentChoiceId>7-B</ParentChoiceId>
    <NodeContent>They sing Luna’s tidy-up song, scoop each napkin together, and the blanket looks neat and ready for more stories.</NodeContent>
  </Node>
  <Node>
    <Id>8-C</Id>
    <Type>choice_outcome</Type>
    <NextNode>7</NextNode>
    <ParentChoiceId>7-C</ParentChoiceId>
    <NodeContent>They sit still while the napkins tumble, the mess grows, and they quickly realize a different plan will feel kinder.</NodeContent>
  </Node>
  <Node>
    <Id>9</Id>
    <Type>linear</Type>
    <NextNode>10</NextNode>
    <NodeContent>With the blanket calm again, Luna opens a pocket-sized sharing story and reads about friends helping during windy days. Milo and Nori lean against her shoulders, sipping warm cocoa while they listen, proud that teamwork kept their picnic cozy.</NodeContent>
  </Node>
  <Node>
    <Id>10</Id>
    <Type>linear</Type>
    <NextNode>END</NextNode>
    <NodeContent>Peach and gold sunset paints the meadow as Luna tucks napkins into the basket and Nori gathers the daisies. They promise another kind adventure tomorrow, strolling home with paws linked and hearts glowing from every caring choice they made together.</NodeContent>
  </Node>
</StoryNodes>
```

Expected output:
```
<InteractiveStorybook>
  <CoverImage>
    <ImagePrompt>Sunrise meadow with Luna bunny arranging a gingham picnic blanket, Milo the fox and Nori the otter hurrying over, warm pastel glow, cheerful smiles.</ImagePrompt>
    <CharacterList>
      <CharacterId>char-luna</CharacterId>
      <CharacterId>char-milo</CharacterId>
      <CharacterId>char-nori</CharacterId>
    </CharacterList>
  </CoverImage>
  <StoryNodes>
    <Node>
      <Id>1</Id>
      <Type>linear</Type>
      <NextNode>2</NextNode>
      <SceneText>Luna packs carrot crackers, berry jam, and tiny cups as sunrise warms the clover path, humming a gentle welcome song.</SceneText>
      <ImagePrompt>Sunrise meadow with Luna bunny carrying a blue basket of treats, golden clover path, joyful expression.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>2</Id>
      <Type>linear</Type>
      <NextNode>3</NextNode>
      <SceneText>She smooths the gingham blanket beside the brook, arranging juice, berries, and little flower vases while she watches for her friends.</SceneText>
      <ImagePrompt>Gingham blanket beside a gentle brook with bowls of strawberries and sparkling juice; Luna bunny adjusting the setup in soft morning light.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>3</Id>
      <Type>choice_question</Type>
      <NextNode>null</NextNode>
      <SceneText>Milo arrives with a rumbling tummy and empty paws, so what friendly plan can Luna offer to help him feel welcome?</SceneText>
      <ImagePrompt>Luna bunny smiling toward Milo fox beside a treat-filled picnic blanket, inviting gesture, bright morning glow.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
      </CharacterList>
      <RetryMessage>Let’s pick a kinder idea.</RetryMessage>
      <Choices>
        <Choice>
          <Id>3-A</Id>
          <IsCorrect>false</IsCorrect>
          <NextNode>4-A</NextNode>
          <ChoiceText>Keep the crackers for herself.</ChoiceText>
        </Choice>
        <Choice>
          <Id>3-B</Id>
          <IsCorrect>true</IsCorrect>
          <NextNode>4-B</NextNode>
          <ChoiceText>Pour the juice together.</ChoiceText>
        </Choice>
        <Choice>
          <Id>3-C</Id>
          <IsCorrect>false</IsCorrect>
          <NextNode>4-C</NextNode>
          <ChoiceText>Ask Milo to search alone.</ChoiceText>
        </Choice>
      </Choices>
    </Node>
    <Node>
      <Id>4-A</Id>
      <Type>choice_outcome</Type>
      <NextNode>3</NextNode>
      <ParentChoiceId>3-A</ParentChoiceId>
      <SceneText>Luna keeps the snacks close, and Milo’s tail droops, reminding her that idea felt unkind.</SceneText>
      <ImagePrompt>Milo fox with drooping tail while Luna clutches the basket on the picnic blanket, gentle caution colors.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>4-B</Id>
      <Type>choice_outcome</Type>
      <NextNode>5</NextNode>
      <ParentChoiceId>3-B</ParentChoiceId>
      <SceneText>Luna and Milo pour the juice side by side, grinning as the picnic feels welcoming.</SceneText>
      <ImagePrompt>Luna bunny and Milo fox pouring juice together on a gingham blanket, bright smiles, sunny meadow.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>4-C</Id>
      <Type>choice_outcome</Type>
      <NextNode>3</NextNode>
      <ParentChoiceId>3-C</ParentChoiceId>
      <SceneText>Sending Milo off alone feels lonely, so Luna pauses, ready to choose a kinder welcome.</SceneText>
      <ImagePrompt>Milo fox stepping away sadly while Luna watches thoughtfully near the picnic spread, soft pastel palette.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>5</Id>
      <Type>linear</Type>
      <NextNode>6</NextNode>
      <SceneText>Working together, Luna passes cups while Milo lines up spoons and hums their tidy-up song.</SceneText>
      <ImagePrompt>Luna bunny handing cups to Milo fox on a neatly arranged picnic blanket, cheerful teamwork mood.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>6</Id>
      <Type>linear</Type>
      <NextNode>7</NextNode>
      <SceneText>Nori arrives with meadow daisies, and the friends cuddle close, whispering thanks for sunshine and crunchy apples.</SceneText>
      <ImagePrompt>Nori otter offering daisies to Luna and Milo beside the brook, joyful smiles, pastel picnic colors.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
        <CharacterId>char-nori</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>7</Id>
      <Type>choice_question</Type>
      <NextNode>null</NextNode>
      <SceneText>A playful breeze lifts napkins and crumbs across the blanket, so what cheerful plan should the friends choose to keep their picnic tidy?</SceneText>
      <ImagePrompt>Napkins swirling above the picnic as Luna, Milo, and Nori reach to catch them, bright midday energy.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
        <CharacterId>char-nori</CharacterId>
      </CharacterList>
      <RetryMessage>Let’s steady our picnic together.</RetryMessage>
      <Choices>
        <Choice>
          <Id>7-A</Id>
          <IsCorrect>false</IsCorrect>
          <NextNode>8-A</NextNode>
          <ChoiceText>Let the napkins flutter away.</ChoiceText>
        </Choice>
        <Choice>
          <Id>7-B</Id>
          <IsCorrect>true</IsCorrect>
          <NextNode>8-B</NextNode>
          <ChoiceText>Sing the tidy-up song together.</ChoiceText>
        </Choice>
        <Choice>
          <Id>7-C</Id>
          <IsCorrect>false</IsCorrect>
          <NextNode>8-C</NextNode>
          <ChoiceText>Sit still and ignore the mess.</ChoiceText>
        </Choice>
      </Choices>
    </Node>
    <Node>
      <Id>8-A</Id>
      <Type>choice_outcome</Type>
      <NextNode>7</NextNode>
      <ParentChoiceId>7-A</ParentChoiceId>
      <SceneText>Letting napkins fly scatters crumbs, so they promise to try again with a kinder plan.</SceneText>
      <ImagePrompt>Napkins blowing into clover while friends look concerned over the messy blanket.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
        <CharacterId>char-nori</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>8-B</Id>
      <Type>choice_outcome</Type>
      <NextNode>9</NextNode>
      <ParentChoiceId>7-B</ParentChoiceId>
      <SceneText>They sing the tidy-up song, gather the napkins, and the blanket looks neat for more fun.</SceneText>
      <ImagePrompt>Friends cheerfully collecting napkins while singing, picnic blanket tidy, bright smiles.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
        <CharacterId>char-nori</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>8-C</Id>
      <Type>choice_outcome</Type>
      <NextNode>7</NextNode>
      <ParentChoiceId>7-C</ParentChoiceId>
      <SceneText>Sitting still lets the mess grow, and the friends agree to choose a better idea.</SceneText>
      <ImagePrompt>Friends seated as napkins tumble around them, thoughtful expressions, gentle breeze.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
        <CharacterId>char-nori</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>9</Id>
      <Type>linear</Type>
      <NextNode>10</NextNode>
      <SceneText>With the blanket calm, Luna reads a sharing story while Milo and Nori sip cocoa against her shoulders.</SceneText>
      <ImagePrompt>Luna bunny reading a tiny book with Milo fox and Nori otter snuggled beside her, warm cocoa cups, serene meadow.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
        <CharacterId>char-nori</CharacterId>
      </CharacterList>
    </Node>
    <Node>
      <Id>10</Id>
      <Type>linear</Type>
      <NextNode>END</NextNode>
      <SceneText>Sunset paints the meadow peach and gold as they pack the basket and promise another caring adventure.</SceneText>
      <ImagePrompt>Friends packing picnic supplies at peach-and-gold sunset, linked paws, daisies gathered, warm glow.</ImagePrompt>
      <CharacterList>
        <CharacterId>char-luna</CharacterId>
        <CharacterId>char-milo</CharacterId>
        <CharacterId>char-nori</CharacterId>
      </CharacterList>
    </Node>
  </StoryNodes>
</InteractiveStorybook>
```


RESPONSE RULES
- Return only the XML snippet. No markdown or commentary.
- Ensure every original node appears once with identical IDs and navigation.
- Do not reintroduce `<NodeContent>` or remove existing elements other than replacing it with `<SceneText>` and `<ImagePrompt>`.
- Always include the `<CoverImage>` element before `<StoryNodes>`.
