# Preface
http://lesscss.org/

# CSS class inherit one or more other classes?

  /*LESS*/
  .rounded_corners {
    border-radius: 8px;
  }

  #header {
    .rounded_corners;
  }

  #footer {
    .rounded_corners;
  }
